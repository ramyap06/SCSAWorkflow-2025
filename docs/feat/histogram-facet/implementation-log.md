# Implementation Log

### 2026-04-22

- Advanced Task CR.3 (formatting cleanup for review readiness) with a narrowed whitespace-only redo.
   - Time: 01:40
   - Restored the affected files to their pre-style-cleanup bytes locally and reapplied only literal trailing-space/trailing-tab deletions so existing line endings stay unchanged.
   - Kept the narrowed CR.3 redo scoped to `src/spac/visualization.py`, `src/spac/utils.py`, and `tests/test_visualization/test_histogram.py`; `src/spac/templates/histogram_template.py` did not require a new local edit in this pass.
   - Verification:
      - `git diff --check -- src/spac/visualization.py src/spac/templates/histogram_template.py src/spac/utils.py tests/test_visualization/test_histogram.py` (clean local working-tree diff)
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac python -m pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q` (53 passed, 1 warning)
   - Remaining for task closure:
      - Land the narrowed cleanup in `HEAD` and then re-run `git diff --check upstream/dev...HEAD`.

- Completed Task CR.2 (non-facet handling of facet-only size hints).
   - Time: 00:49
   - Updated `histogram()` so `facet_fig_width` / `facet_fig_height` are popped early and only parsed/validated when `facet=True`.
   - Kept non-facet direct calls ignoring facet-only size hints silently, while preserving paired-hint validation in actual facet mode.
   - Kept the regression coverage compact with one focused non-facet no-op test near the existing facet figure-size-hint tests.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac python -m pytest tests/test_visualization/test_histogram.py -q` (45 passed)

- Completed Task CR.1 (documentation/review alignment for `max_groups` and `facet_ncol` direct-call edge cases).
   - Time: 00:12
   - Reclassified the former strict-validation follow-up as docs/polish only, keeping current `histogram()` direct-call coercion behavior unchanged for this PR.
   - Updated the `histogram()` `**kwargs` docstring wording to describe `max_groups` and `facet_ncol` behavior without over-promising stricter integer-only runtime validation.
   - Synced task, overview, decision, and review records so Finding 1 remains historical but is now marked as a non-blocking docs/polish disposition.
   - Verification:
      - Static review of `src/spac/visualization.py`, `local/docs/feat/histogram-facet/task-details.md`, `local/docs/feat/histogram-facet/overview.md`, `local/docs/feat/histogram-facet/decisions.md`, and `local/docs/feat/histogram-facet/code-review-2026-04-21.md`.

### 2026-04-21

- Refined grouped shared-bin handling after Task 18 closure.
   - Kept categorical shared-slot padding inside `build_grouped_histogram_table` while continuing to reuse shared numeric bin edges for grouped/facet plotting.
   - Restricted Rice-rule auto-bin fallback to numeric data so categorical default-like `bins` no longer trigger meaningless auto-bin computation.
   - Folded facet return-data assertions into existing default/shared-bin tests and removed dead histogram-test imports.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q` (52 passed, 1 warning)

- Completed Task 18 with the finalized count-table consistency design.
   - Moved shared-bin derivation into the grouped histogram-table helper.
   - Switched facet plotting to the same precomputed grouped histogram-bin count table pattern already used by the single-plot and `together=True` branches.
   - Preserved the count-table output contract and deferred broader raw-data/KDE fidelity questions to future work.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q` (54 passed, 1 warning)

- Advanced Task 18 implementation for review.
   - Reused one grouped histogram-bin table builder across grouped-overlay and facet grouped branches in `histogram()`.
   - Changed facet return `df` to grouped histogram-bin data with `group_by` metadata instead of raw `plot_data`.
   - Added focused numeric and categorical facet return-data contract tests.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q` (54 passed, 1 warning)

- Refined histogram facet unittest readability after Task 19 closure.
   - Kept the new numeric-annotation facet smoke coverage and simplified the test structure around it.
   - Extracted only the reusable long-label facet fixture helper, while keeping one-off shared-bin setups inline for easier review.
   - Preserved the dedicated `_derive_facet_geometry` helper tests after review instead of folding that cleanup into this histogram-test refinement.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q` (50 passed, 1 warning)

- Completed Task 19 and closed the remaining Task 7 coverage item.
   - Added one thin facet smoke test for numeric annotations sourced from `adata.obs` using local test setup only.
   - Verified the current histogram/helper/template suite covers the introduced facet logic paths without adding broader template or layout-fragile tests.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q` (52 passed, 1 warning)

- Completed Task 20 (plotting-control validation simplification).
   - Removed template-side allow-list validation for `multiple`, `element`, and `stat`.
   - Stopped coercing grouped-separate `multiple` to `"dodge"`; template now forwards `multiple` only for grouped same-axis overlays.
   - Added defensive grouped non-overlay cleanup in `histogram()` so direct calls ignore irrelevant `multiple`.
   - Added focused regression coverage showing `multiple="fill"` no longer crashes grouped-separate rendering.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (43 passed)
      - `NUMBA_CACHE_DIR=/tmp/numba_cache MPLCONFIGDIR=/tmp/mplconfig XDG_CACHE_HOME=/tmp/.cache conda run -n spac pytest tests/templates/test_histogram_template.py -q` (1 passed, 1 warning)

- Completed Task 21 (template validation convention audit and alignment at histogram-PR scope).
   - Reviewed histogram template/core validation changes against `upstream/dev` and confirmed the remaining mixed token/typed parsing pattern is broader template convention rather than a histogram-only regression.
   - Locked the current PR boundary: keep histogram-introduced token/helper behavior aligned, document strict template-token expectations where needed, and defer repo-wide template cleanup beyond this workstream.
   - Verification: audit of `src/spac/templates/histogram_template.py`, `src/spac/templates/template_utils.py`, and `src/spac/visualization.py` against current plan scope and `upstream/dev`.

### 2026-04-20

- Advanced Task 21 (`_parse_optional_number` helper design resolution).
   - Refactored `_parse_optional_number` in `histogram()` to a smaller shared-mechanics helper that handles only defaulting, explicit token lookup, numeric coercion, and finite/positive checks.
   - Kept special token policy explicit at call sites (`"unlimited"` for `max_groups`; `""`/`"auto"`/`"none"` for `facet_ncol`) and slightly clarified the helper docstring.
   - Resolved the previously tracked `_parse_optional_number` open issue without changing current behavior contracts.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (42 passed)
      - `NUMBA_CACHE_DIR=/tmp/numba_cache XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp conda run -n spac pytest tests/templates/test_histogram_template.py -q` (1 passed, 1 warning)

- Completed Task 16 (facet figure title layout handling).
   - Replaced unconditional facet layout behavior in template with a concise row-scaled `tight_layout(...)` rule so 1/2/3/4+ facet-row grids reserve different spacing.
   - Tuned top/left/bottom/right spacing policy for dense and compact facet grids while preserving current visual expectations from notebook checks.
   - Removed unused `suptitle_artist` variable from template cleanup after final layout refactor.
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp conda run -n spac pytest tests/templates/test_histogram_template.py -q` (1 passed, 1 warning)
      - `NUMBA_CACHE_DIR=/tmp/numba_cache XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (42 passed)

- Completed Task 17 (grouped `group_by` max-group guardrail validation).
   - Added grouped-mode guardrail validation in `histogram()` based on non-null unique group count.
   - Wired template `Max_Groups` through to core and normalized `Group_by` token handling at the template boundary.
   - Finalized current contract in implementation: default threshold `20`, positive-int override, `"unlimited"` bypass, and explicit `None` resolving to default-threshold behavior.
   - Simplified new max-group tests to concise behavior-level coverage (default guardrail, explicit override, unlimited bypass, explicit-None default behavior, representative invalid values).
   - Verification:
      - `NUMBA_CACHE_DIR=/tmp/numba_cache XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp/mpl conda run -n spac pytest tests/test_visualization/test_histogram.py -q -k max_groups` (5 passed)
      - `NUMBA_CACHE_DIR=/tmp/numba_cache XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp/mpl conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (42 passed)

- Completed Task 11 (facet long x-label layout handling).
   - Added `facet_tick_rotation` hint wiring from template into histogram facet geometry.
   - Implemented a tick-length/rotation-based geometry heuristic in `_derive_facet_geometry` so long categorical labels expand default facet height and slightly tighten aspect when explicit size hints are absent.
   - Kept explicit facet figure-size hints authoritative and allowed facet template `Figure_Width` / `Figure_Height` to remain `"auto"` so core geometry can size the figure automatically.
   - Added focused histogram regressions for zero-rotation parity, long-label auto sizing, and explicit-size precedence.
   - Verification: committed updates in `tests/test_visualization/test_histogram.py` cover the final Task 11 behavior.

### 2026-04-19

- Updated plan state to match current user-directed code/test choices.
   - Reopened Issues 1-3 and rolled Task 7 / Task 11 back to in-progress status.
   - Removed unapproved decision-log entries related to deferral/completion claims.
   - Documented the current template-side correction that rotates x tick labels rather than the figure-level x label, while keeping the broader long-label layout problem unresolved.

- Advanced Task 11 foundation (facet rotation presentation fix).
   - Corrected facet template rotation handling to rotate tick labels rather than the figure-level x label.
   - This became the stable presentation baseline for the later long-label geometry work completed on 2026-04-20.

- Completed Task 15 (facet layout hint validation simplification).
   - Replaced generic facet-hint normalization in `histogram` with explicit local parsing/validation for `facet_ncol`, `facet_fig_width`, `facet_fig_height`, and `facet_tick_rotation`.
   - Updated `_derive_facet_geometry` tests to match the pre-normalized helper contract and moved validation expectations to histogram-level coverage.
   - Removed `normalize_positive_number` and deleted obsolete utils coverage once the facet validation ownership was finalized.
   - Verification: committed updates in `tests/test_visualization/test_histogram.py` and `tests/test_visualization/test_derive_facet_geometry.py` reflect the final fail-fast contract.

### 2026-04-18

- Advanced Task 7 / Task 11 exploration, later partially reverted by user direction.
   - Added categorical facet `bins`-ignore regression coverage and an initial long-label handling attempt before final user approval.
   - This checkpoint should not be treated as final completion for either task.
   - Verification at that checkpoint:
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac python -m pytest tests/test_visualization/test_histogram.py tests/templates/test_histogram_template.py -q` (35 passed)

- Completed Task 14 (core input normalization refactor).
   - Added `normalize_positive_number` in `src/spac/utils.py` as the shared core positive-number normalization helper with logging for default-like, invalid, and non-positive inputs.
   - Refactored `_derive_facet_geometry` to use the shared helper and kept automatic `facet_ncol` selection logging in visualization.
   - Kept `text_to_others` backward-compatible and added direct unit coverage for both the new helper and wrapper behavior.
   - Verification:
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/test_utils/test_normalize_positive_number.py tests/test_utils/test_text_to_others.py -q`
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/test_visualization/test_derive_facet_geometry.py -q`
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/test_visualization/test_histogram.py -q`
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/templates/test_histogram_template.py -q`

- Completed Task 13 (facet geometry helper contract clarification + dedicated helper tests).
   - Expanded `_derive_facet_geometry` docstring to explicitly define normalization, automatic column selection, figure-size-hint handling, and aspect/panel-size guardrails.
   - Added `tests/test_visualization/test_derive_facet_geometry.py` with focused coverage for `"auto"`, explicit positive integer columns, fallback sanitization, figure-size-derived geometry, and `facet_ncol` clamping.
   - Verification:
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/test_visualization/test_derive_facet_geometry.py -q`
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/test_visualization/test_histogram.py -q`
      - `PYTHONPATH=src XDG_CACHE_HOME=/tmp MPLCONFIGDIR=/tmp NUMBA_CACHE_DIR=/tmp/numba_cache conda run -n spac pytest tests/templates/test_histogram_template.py -q`
   - Identified shared input-normalization utility extraction as follow-up work, which was later implemented in Task 14.
   - Verification:
      - `pytest tests/test_visualization/test_histogram.py -q` (32 passed)

- Reordered facet layout-hint tests to follow facet smoke/core behavior flow.
   - Moved `test_facet_ncol_layout_hints` and `test_facet_figure_size_hints` to a later position after smoke/title/label/categorical facet tests.
   - Updated plan sequencing to a commit-first checkpoint before next implementation.
   - Expanded Task 13 scope to include helper docstring/API-contract improvements plus independent helper unittest coverage.
   - Verification:
      - `pytest tests/test_visualization/test_histogram.py -q` (32 passed)

- Revised Task 4/7 facet `facet_ncol` coverage scope per docstring contract.
   - Updated `test_facet_ncol_layout_hints` to explicitly assert positive-int layout and documented `"auto"` behavior.
   - Kept one lightweight invalid-input fallback assertion (`"bad"` -> auto layout) and removed non-essential float-like coverage.

- Completed Task 4 (facet layout precedence/docs + parameter test coverage).
   - Added concise template comments documenting facet geometry precedence/formula and final output-size ownership.
   - Added histogram facet layout tests for `facet_ncol` valid/invalid/type behavior and `facet_fig_width`/`facet_fig_height` checks.
   - Tightened layout-hint normalization for facet figure size hints to safely sanitize invalid/non-positive values in direct plotting API use.
   - Verification:
      - `conda activate spac && python -m pytest tests/test_visualization/test_histogram.py -q` (32 passed)
      - `conda activate spac && python -m pytest tests/templates/test_histogram_template.py -q` (1 passed, 1 warning)

- Completed Task 5 (default-like bins fallback regression coverage).
   - Added/kept focused fallback regression coverage for explicit default-like `bins` input (`None`) and validated Rice-rule behavior remains active.
   - Verification: `conda activate spac && python -m pytest tests/test_visualization/test_histogram.py -q` (30 passed).

- Completed Task 1 (shared-bin regression coverage + shared-scale assertions).
   - Added facet numeric shared-bin consistency test using integer/default-like bins inputs (`4`, `None`) to confirm shared centers/limits/ticks across facet panels.
   - Added facet categorical shared-bin consistency test on unbalanced groups and verified shared x tick positions/labels and shared y ticks across facet panels.
   - Ran a temporary proof test in non-facet grouped mode (`together=False`, `facet=False`) to confirm the same shared-bin-center assertion fails there; removed the temporary test after verification.
   - Verification: `conda activate spac && python -m pytest tests/test_visualization/test_histogram.py -q` (30 passed).

### 2026-04-17

- Completed Task 10 (helper boundary relocation and naming alignment).
   - Renamed `_parse_histogram_layout_kwargs` to `_parse_facet_layout_hints`.
   - Moved `compute_global_bin_edges` and `resolve_hist_axis_labels` into `histogram` local scope.
   - Updated local helper docstrings to compact NumPy-style format.
   - Verification: `conda activate spac && python -m pytest tests/test_visualization/test_histogram.py -q` (27 passed).

- Advanced Task 7 (remaining visualization test coverage in current scope).
   - Added facet validation tests (`facet=True` requires `group_by`; conflict with `together=True`).
   - Kept one minimal annotation-based categorical facet regression test.
   - Deferred `shrink`/`alpha` unittest coverage by scope decision.
   - Verification: `conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (27 passed).

- Extended template I/O coverage for facet-mode parameters.
   - Added facet-mode parameter wiring to `tests/templates/test_histogram_template.py` while keeping the test I/O-focused.
   - Verification: `conda run -n spac pytest tests/templates/test_histogram_template.py -q` (1 passed, 1 warning)

- Completed Task 8 (template-side facet x-label ownership refactor).
   - Removed duplicate semantic x-label reassignment in `src/spac/templates/histogram_template.py`; template now applies presentation-only rotation to the existing figure-level x-label.
   - Verification:
      - `conda run -n spac pytest tests/templates/test_histogram_template.py -q` (1 passed, 1 warning)
      - `conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (24 passed at that checkpoint)

### 2026-04-16

- Completed Task 12 (facet-test decomposition + smoke-path contract).
   - Split heavy facet coverage into focused structure/title/label checks and a thin smoke-path test.
   - Verification: `conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (24 passed at that checkpoint).

- Completed Task 6 (facet label strategy + stat mapping test alignment).
   - Updated facet assertions to figure-level label policy (empty per-axis labels in facet mode).
   - Added non-default `stat='density'` y-label regression assertion.
   - Verification: `conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (22 passed at that checkpoint).

- Completed Task 9 (facet external-`ax` guardrail + figure lifecycle cleanup).
   - Added external-`ax` validation for unsupported grouped-separate/facet layouts.
   - Tightened figure ownership/closure behavior in `histogram`.
   - Added regression coverage in `tests/test_visualization/test_histogram.py` for external-`ax` support/rejection modes.
   - Verification: focused unittest runs for external-`ax` modes in `spac` environment.
