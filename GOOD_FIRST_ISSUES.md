# Good First Issues

Bite-sized, well-scoped tasks for new contributors. Each one touches a small
surface area and has a clear definition of done.

---

## 1. Dead variable in `precompute.py`

**Difficulty:** trivial · **Files:** `precompute.py`

`id_batch` is appended to and reset alongside `blob_batch` but never read.
Remove it.

**Done when:** `grep id_batch precompute.py` returns nothing and the script
still runs.

---

## 2. Add a `.dockerignore`

**Difficulty:** trivial · **Files:** `.dockerignore` (new)

`Dockerfile` does `COPY . .`, which copies the multi-GB `venv/`, the ~487 MB
`data/` directory, `artifacts/` and `.git` into the build context and image.
Add a `.dockerignore` excluding `venv/`, `.git/`, `artifacts/`, `output/`,
`__pycache__/` and `data/` (data is volume-mounted at runtime per the README).

**Done when:** `docker build` context is a few MB instead of several GB.

---

## 3. Stale performance numbers in README

**Difficulty:** trivial · **Files:** `README.md`

The Performance table still says "Ranking & Validation ~1.5 min". After the
batch-loading fix it completes in roughly 5 seconds (2.2 s rank + validation).
Re-measure with `python rank.py` and update the table.

**Done when:** the table matches a fresh local run.

---

## 4. Unreachable keywords in `TIER_2_INSTITUTIONS`

**Difficulty:** easy · **Files:** `config.py`, `signals.py`

`TIER_1_INSTITUTIONS` contains `"iiit"` and is checked first, so the
`"iiit"`/`"iiitm"` entries in `TIER_2_INSTITUTIONS` can never match. Same for
`"nit"` (tier-1 has `"nit "`, tier-2 has `"nit"` — the unspaced tier-2 version
also substring-matches words like "monitoring"). Deduplicate the lists and add
word-boundary matching in `_check_education_tier`'s keyword fallback.

**Done when:** no keyword appears in both lists and `"monitoring university"`
doesn't classify as tier-2.

---

## 5. Crash-proof `build_outreach_pack` name handling

**Difficulty:** easy · **Files:** `app.py`

`name.split()[0] if name else "there"` raises `IndexError` when
`anonymized_name` is whitespace-only (`" ".split()` is `[]`). Use
`name.split()[0] if name.split() else "there"` or equivalent.

**Done when:** a candidate dict with `anonymized_name: "  "` produces a
greeting of "there" instead of crashing the Export tab.

---

## 6. Defend against `null` role descriptions in `compute_technical_fit`

**Difficulty:** easy · **Files:** `signals.py`

`compute_technical_fit` builds its text blob with `r.get("description", "")`,
which returns `None` (not `""`) when the JSON value is `null`, crashing
`" ".join(...)`. Other call sites already use `(r.get("description", "") or "")`.
Apply the same guard here (and to the `headline`/`summary`/`current_title`
concatenation).

**Done when:** a candidate with `"description": null` scores without raising.

---

## 7. Demographics cache never invalidates

**Difficulty:** easy · **Files:** `app.py`

`pool_demographics()` writes `artifacts/demographics.csv` once and returns it
forever, even if `data/candidates.jsonl` changes. Compare the mtime of the
cache file against the JSONL and rebuild when stale.

**Done when:** touching `candidates.jsonl` causes a rebuild on next app start.

---

## 8. Add a pytest regression suite

**Difficulty:** medium · **Files:** `tests/` (new), `requirements.txt`

All scoring edge cases are currently verified ad hoc. Create
`tests/test_signals.py`, `tests/test_disqualify.py`, `tests/test_evidence.py`,
`tests/test_engine.py` covering at minimum:

- "version control" does **not** trigger the CV/robotics penalty
- all-empty company names do **not** zero `career_quality`
- "Senior Principal Engineer" → seniority level 4; "International Sales Lead" → 3
- newest-first career history still detects upward progression
- `tier_1`/`tier_2` education strings parse
- `generate_reasoning` only claims production evidence when present
- `mmr_rerank` returns unique indices and starts with the top-scored candidate
- `top_k_indices` ordering is deterministic on ties

**Done when:** `pytest` passes and runs in under 30 s without artifacts.

---

## 9. Export CSV tie-breaking doesn't match the challenge spec

**Difficulty:** medium · **Files:** `engine.py`, `app.py`

The spec requires ties (at 3 displayed decimals) to be broken by ascending
`candidate_id`. `rank.py` does this, but the dashboard's Export tab uses
`engine.top_k_indices`, which orders by score rounded to 6 decimals and
tie-breaks by array index. Align the export path with the spec (round to 3
and tie-break by id).

**Done when:** the dashboard-exported CSV passes
`python data/validate_submission.py <file>` including tie-break ordering for
synthetic tied scores.
