# Histogram Facet Code Review

Date: 2026-04-21

Scope:
- Branch under review: `feat/histogram-facet`
- Target branch: `upstream/dev`
- Diff reference: `git diff upstream/dev...HEAD`

## Findings

1. `[medium]` `src/spac/visualization.py:629`, `src/spac/visualization.py:758` - Non-integral direct-call inputs are silently truncated instead of rejected.
The `histogram()` docstring says `facet_ncol` and `max_groups` are positive integers (or a documented keyword), but `_parse_optional_number()` currently uses `int(value)` for numeric inputs. In direct calls this accepts values like `facet_ncol=2.7` and applies a 2-column layout, and it turns `max_groups=2.7` into threshold `2` while accepting `max_groups=3.9`. That makes invalid input change behavior instead of failing fast.

2. `[medium]` `src/spac/visualization.py:635`, `src/spac/visualization.py:819` - Facet-only layout hints currently error even when facet mode is off.
The docstring presents `facet_fig_width` and `facet_fig_height` as facet-mode kwargs, but the parser validates them unconditionally and raises on direct calls like `histogram(..., facet=False, facet_fig_width=8)`. That makes shared kwargs brittle and conflicts with the stated non-facet guardrail goal that facet layout hints should not leak into non-facet plotting paths.

3. `[medium]` `src/spac/visualization.py:539`, `src/spac/templates/histogram_template.py:95` - New user-facing functionality lacks corresponding public docs updates.
This branch adds public histogram/template controls such as `facet`, `facet_ncol`, `max_groups`, figure-size hints, and additional plotting controls, but `git diff --name-only upstream/dev...HEAD` contains no README or user-doc updates. `CONTRIBUTING.md` says: `If the pull request adds functionality, the docs should be updated.`

4. `[medium]` `src/spac/templates/histogram_template.py:158`, `tests/templates/test_histogram_template.py:33` - The template boundary still lacks targeted validation tests.
`run_from_json()` now owns strict user-facing validation for bins, `Max_Groups`, figure size, `Facet`, `Facet_Ncol`, and grouped/facet logical consistency, but the template test file still has only one happy-path I/O test. That leaves most of the changed template-side validation and handled-exception branches uncovered, which is weaker than the `CONTRIBUTING.md` unittest guidance to trigger handled exceptions and aim for comprehensive coverage.

5. `[low]` `src/spac/visualization.py:29`, `src/spac/templates/histogram_template.py:236`, `tests/test_visualization/test_histogram.py:976` - The patch currently fails formatting checks.
`git diff --check upstream/dev...HEAD` reports trailing whitespace in touched files, so the diff does not currently satisfy the repo requirement to conform to formatting expectations before submission.

## Follow-Up Disposition Notes

- Finding 1 was reclassified on 2026-04-22 as docs/polish only for this PR.
  - Keep current direct-call coercion behavior unchanged.
  - Treat the float-like direct-call cases as edge-case API behavior rather than a must-fix contract break.
  - Relax the `histogram()` docstring wording so it no longer over-promises stricter direct-call integer-only validation than the implementation enforces.

## Checks Performed

- Reviewed the diff for:
  - `src/spac/visualization.py`
  - `src/spac/templates/histogram_template.py`
  - `src/spac/utils.py`
  - `tests/test_visualization/test_histogram.py`
  - `tests/test_visualization/test_derive_facet_geometry.py`
  - `tests/templates/test_histogram_template.py`
- Reviewed the local feature docs in `local/docs/feat/histogram-facet/`
- Ran:
  - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
  - Result: `52 passed, 1 warning`
- Ran:
  - `git diff --check upstream/dev...HEAD`
  - Result: reports trailing whitespace in touched files
- Reproduced direct-call edge cases:
  - `histogram(..., group_by='g', facet=True, facet_ncol=2.7)` was accepted and produced a 2-column facet layout
  - `histogram(..., group_by='g', together=True, max_groups=2.7)` raised with threshold `2`
  - `histogram(..., group_by='g', together=True, max_groups=3.9)` was accepted
  - `histogram(..., facet=False, facet_fig_width=8)` raised `ValueError`

## Open Questions / Assumptions

- Assumed the intended target branch is the local ref `upstream/dev`.
- Review scope was limited to the current diff and its directly related tests/docs, not a full re-audit of unchanged histogram callers.
