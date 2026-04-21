# Task Details

### Task 1. Shared Global Bins Helper
Location: src/spac/visualization.py

Status: Done

Action items:
- [x] Create a shared bin-edge helper used by both together and facet grouped flows (currently histogram-local: `compute_global_bin_edges`).
- [x] Replace together-mode duplicated bin-edge logic with helper call.
- [x] Replace facet-mode duplicated bin-edge logic with helper call.
- [x] Keep helper documented with concise behavior notes.
- [x] Add shared-bin regression tests here, explicitly covering both numerical and categorical input cases.
- [x] Add facet shared-scale assertions for y-tick consistency across facet panels (numeric and categorical paths).

### Task 2. Histogram/Template Parameter Boundary
Location: src/spac/templates/histogram_template.py

Status: Done

Action items:
- [x] Expose facet and facet_ncol at template layer.
- [x] Pass target_fig_width and target_fig_height as internal hints.
- [x] Keep facet_vertical_threshold/facet_height/facet_aspect out of public API contract.

### Task 3. Grouped Annotation Title Bugfix
Location: src/spac/visualization.py

Status: Done

Action items:
- [x] Restore per-group annotation title assignment in non-facet grouped mode.
- [x] Verify grouped annotation titles render correctly.

### Task 4. Facet Layout Derivation and Non-Facet Kwarg Guardrails
Location: src/spac/visualization.py, src/spac/templates/histogram_template.py, tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Derive facet geometry from figure target size plus grid shape.
- [x] Add panel-size/aspect guardrails.
- [x] Facet layout kwargs must not leak into non-facet seaborn calls.
- [x] Lock decision for this PR: keep current `_derive_facet_geometry` ratio approximation.
- [x] Document facet geometry precedence and formula in template doc/comments.
- [x] Add unit tests for facet layout parameters, including `facet_ncol` documented contract (`"auto"`, positive int) and `facet_fig_width`/`facet_fig_height` checks.

### Task 5. Bins Default-Like Fallback Policy and Behavior
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py

Status: Done

Action items:
- [x] Keep template as user-facing bins policy entrypoint.
- [x] Keep explicit user bins unchanged.
- [x] Normalize bare histogram default-like bins inputs to Rice-rule fallback (missing/None/auto-like).
- [x] Keep facet numeric shared-bin behavior for cross-facet consistency.
- [x] Add focused regression tests for default-like bins fallback behavior.

### Task 6. Facet Label Strategy and Test Alignment
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Implementation plan:
1. Replace per-axis xlabel/ylabel assertions with figure-level label assertions:
   - assert per-axis labels are empty in facet mode,
   - assert `fig._supxlabel` and `fig._supylabel` are present and match expected text.
2. Add one focused regression assertion for label strategy under non-default stat (e.g., `stat='density'`) to confirm y super-label reflects stat mapping.
3. Run only updated tests first for fast feedback, then run full histogram test file.
4. If needed, do minimal assertion tuning to avoid over-constraining matplotlib internals.

Action items:
- [x] Lock decision: facet mode uses figure-level labels.
- [x] Keep per-axis labels empty in facet mode and set supxlabel/supylabel.
- [x] Update test_facet_plot to assert figure-level labels.
- [x] Add one histogram-level regression assertion to lock label strategy and stat mapping.
- [x] Keep facet label/title test coverage for the figure-level labeling contract under this task.

### Task 7. Visualization Histogram Unit-Test Completion
Location: tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Run focused histogram test file and make it fully green.
- [x] Complete remaining histogram coverage after Task 12 decomposition/smoke-path work.
- [x] Add/adjust facet validation unit tests (`facet=True` requires `group_by`, and `facet` with `together=True` conflict).
- [x] Decide not to add dedicated unittests or extra custom validation for seaborn passthrough kwargs such as `shrink` and `alpha` in this PR.
- [x] Add one focused facet regression test for annotation-based categorical data using lightweight assertions.
- [x] Double-check annotation-based categorical regression test scope/assertions to ensure it remains minimal and input-difference-focused.
- [x] Decide to add one thin numeric-annotation facet smoke unittest and track its implementation in Task 19.
- [x] Simplify the current categorical facet `bins`-ignore regression coverage so it matches the intended lightweight scope.
- [x] Decide to move plotting-control validation scope cleanup into Task 20.
- [x] Verify all introduced facet logic paths are covered after Tasks 17 and 19 are settled.

### Task 8. Template Histogram Unit-Test Completion
Location: tests/templates/test_histogram_template.py

Status: Done

Action items:
- [x] Verify template output contract remains stable (`saved_files` keys and generated artifacts).
- [x] Remove duplicate facet x-label reassignment in template and keep semantic x-label ownership in `histogram`.
- [x] Run template test file and verify passing status with histogram test updates.

### Task 9. Facet `ax` Guardrail and Figure Lifecycle Refactor
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Reject `facet=True` when external `ax` is provided, with a clear validation message.
- [x] Refactor histogram figure creation/closure flow to avoid throwaway figures and keep strict internal-ownership closing.
- [x] Add minimal regression tests for the guardrail and lifecycle behavior.

### Task 10. Helper Boundary Relocation and Naming Alignment (Facet Scope)
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Agreed renaming plan (to execute in this task):
- `_parse_histogram_layout_kwargs` -> `_parse_facet_layout_hints` (universal helper naming).
- `compute_global_bin_edges` / `resolve_hist_axis_labels` references in this plan correspond to current helpers `_compute_global_bin_edges` / `_resolve_histogram_axis_labels` in code.
- Keep `_derive_facet_geometry` name unchanged.

Action items:
- [x] Rename `_parse_histogram_layout_kwargs` to `_parse_facet_layout_hints`.
- [x] Move `compute_global_bin_edges` and `resolve_hist_axis_labels` logic into `histogram` scope.
- [x] Verify histogram-level tests after helper-boundary relocation under CONTRIBUTING.md expectations.

### Task 11. Facet Long X-Label Layout Handling
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py, tests/test_visualization/test_histogram.py, tests/templates/test_histogram_template.py

Status: Done

Implementation decision:
- Use geometry-aware default facet sizing in the core histogram geometry path when explicit figure-size hints are absent.
- Keep the template passing lay-out hints like `facet_fig_width` and `facet_fig_height`, and presentation-only label rotation.
- Keep axis-label abbreviation out of this PR and treat it as future work.

Action items:
- [x] Apply facet rotation to tick labels in template post-processing rather than rotating the figure-level x label.
- [x] Extend `_derive_facet_geometry` with a tick-label burden heuristic that increases facet height and rebalances aspect when long rotated labels would otherwise shrink the plot area too far.
- [x] Keep explicit figure-size hints authoritative while allowing facet template figure size to remain `"auto"` and defer to derived geometry.
- [x] Add a small, low-noise log message when automatic facet sizing reacts to unusually long labels.
- [x] Add appropriate unittests for zero-rotation parity, long-label auto sizing, and explicit-size precedence.

### Task 12. Facet Plot Test Decomposition and Smoke-Path Contract
Location: tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Split the current heavy facet test into focused tests for structure, title mapping, label policy, and rendering smoke-path checks.
- [x] Add one thin facet smoke-path test that verifies end-to-end execution and basic plotted output presence, including lightweight bar-level presence checks (non-empty facet patches/artists).
- [x] Add split assertions that validate facet behavior using stable, non-geometry-sensitive checks.
- [x] Run focused facet tests first, then run the full histogram test file.

### Task 13. Facet Geometry Helper API/Docstring + Independent Unittests
Location: src/spac/visualization.py, tests/test_visualization/test_derive_facet_geometry.py

Status: Done

Action items:
- [x] Improve `_derive_facet_geometry` docstring so documented behavior is explicit, API-directed, and testable.
- [x] Add a dedicated unittest file for `_derive_facet_geometry` as an independent helper-function test target.
- [x] Cover documented/default-like and corner-case helper inputs: positive integer `facet_ncol`, `"auto"`, fallback behavior, and figure-size hint sanitization.
- [x] Keep helper-level assertions focused on deterministic documented outputs (ncol/height/aspect and normalized size-hint outputs).
- [x] Run focused helper test file and keep existing histogram integration tests green.

### Task 14. Core Input Normalization Refactor
Location: src/spac/utils.py, src/spac/visualization.py, tests/test_utils/test_normalize_positive_number.py, tests/test_utils/test_text_to_others.py

Status: Done

Action items:
- [x] Add a reusable core positive-number normalization helper with explicit logging for fallback cases.
- [x] Refactor `_derive_facet_geometry` to use the core helper while keeping facet auto-selection logging local.
- [x] Keep `text_to_others` behavior backward-compatible and verify existing numeric/default conversions still work.
- [x] Add direct unit tests for the new helper and preserve wrapper regression coverage.

### Task 15. Facet Layout Hint Validation Simplification
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py, tests/test_visualization/test_derive_facet_geometry.py, tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Keep template strict about user-facing `Facet_Ncol` and figure-size inputs, including paired facet figure hints.
- [x] Replace generic facet-hint normalization in `histogram` with explicit local parsing/validation for `facet_ncol`, `facet_fig_width`, `facet_fig_height`, and `facet_tick_rotation`.
- [x] Keep `_derive_facet_geometry` as a pure geometry helper that consumes pre-normalized inputs.
- [x] Remove `normalize_positive_number` and obsolete unit coverage once the facet validation contract is finalized.

### Task 16. Facet Figure Title Layout Handling
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py

Status: Done

Implementation remarks:
- The current overlap is a template/core interaction problem, not a facet panel-geometry problem alone.
- `histogram` already owns facet panel geometry and figure-level axis labels, while the template owns the figure-level title.
- Recommended implementation path: keep title ownership in the template, but replace the current unconditional `plt.tight_layout()` path with facet-aware figure layout handling that explicitly reserves top margin for `fig.suptitle(...)`.

Action items:
- [x] Replace the current unconditional template layout call with facet-aware layout handling that preserves top room for figure titles on dense facet grids.
- [x] Verify the fix on a many-facet example (for example, ~14 facets) and ensure the plotting panels remain readable.
- [x] Add or update the minimal tests or manual verification notes needed for this template-level layout change.

### Task 17. Grouped `group_by` Max-Group Guardrail Validation
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Implementation remarks:
- The key failure mode is excessive group cardinality, not only dtype: even categorical labels can produce unreadable or unstable grouped plots when group count is too large.
- Apply one grouped-mode guardrail across facet/together/separate grouped paths.
- Use a default maximum group count (`max_groups=20`), allow explicit positive-int override, and allow disabling with `max_groups="unlimited"`.
- Keep direct-call handling concise: explicit `max_groups=None` resolves to default threshold behavior.
- Validation should fail fast with a clear `ValueError` message only (no additional warning) that reports observed groups, threshold, and how to override.

Action items:
- [x] Add grouped-mode guardrail validation in `histogram()` that raises when non-null unique `group_by` count exceeds `max_groups` (default `20`).
- [x] Support `max_groups` override in `histogram()` (`int > 0`) and `"unlimited"` bypass, while resolving explicit `None` to default threshold behavior.
- [x] Expose optional `Max_Groups` in template input handling and pass it through to `histogram()`.
- [x] Add focused histogram tests for: default-threshold rejection, explicit larger-threshold success, `"unlimited"` bypass behavior, explicit `max_groups=None` default-threshold behavior, and representative invalid inputs.

### Task 18. Facet Histogram Download DataFrame Contract
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Implementation remarks:
- Keep this task narrow: align facet with the existing histogram count-table pattern used by the single-plot and `together=True` branches.
- Facet should plot and return the same grouped histogram-bin count table with grouping metadata, not raw `plot_data`.
- Do not redesign full seaborn/raw-data fidelity in this task.

Action items:
- [x] Refactor the grouped histogram-table helper so it owns shared-bin derivation for grouped count-table paths.
- [x] Use that grouped histogram-bin table for facet plotting as well as facet return `df`.
- [x] Keep Task 18 output count-based; do not add normalized/KDE-specific output columns in this PR.
- [x] Preserve current facet shared-bin behavior for numeric and categorical cases under the precomputed-table path.
- [x] Add focused tests for numeric and categorical facet return-data contracts under the count-table pattern.

### Task 19. Numeric-Annotation Facet Smoke Coverage
Location: tests/test_visualization/test_histogram.py

Status: Done

Implementation remarks:
- Numeric annotation is user-exposed and differs from numeric feature mainly at the data-source boundary (`adata.obs` instead of feature/layer selection).
- Current coverage already exercises numeric feature facet mode and categorical annotation facet mode, so this should stay as one thin contract test rather than a heavy rendering test.
- Recommended test shape: add one numeric annotation column to the histogram test fixture (or local test setup), call `histogram(annotation=..., group_by=..., facet=True)`, and assert only basic end-to-end behavior such as returned structure, expected facet count, and plotted bar presence.

Action items:
- [x] Add one numeric annotation input to the histogram test fixture or local test setup.
- [x] Add one thin facet smoke unittest for numeric annotation sourced from `adata.obs`.
- [x] Assert only structure, facet count, and non-empty plotted output.

### Task 20. Plotting-Control Validation Simplification
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Implementation remarks:
- Template-side explicit allow-list validation for `multiple`, `element`, and `stat` is not worth keeping when SPAC is not intentionally defining a narrower contract than seaborn.
- `multiple` is only meaningful for grouped histograms drawn on the same axes.
- Grouped-separate and facet grouped paths should ignore `multiple` instead of coercing it to `"dodge"` or raising a new SPAC-only error.
- Since `histogram()` is also publicly callable, keep a matching defensive cleanup there for direct calls.

Action items:
- [x] Remove template-side explicit allow-list validation for `multiple`, `element`, and `stat`.
- [x] Stop coercing grouped-separate `multiple` to `"dodge"` in the template.
- [x] Pass `multiple` only for grouped same-axis overlays from the template.
- [x] Add matching defensive cleanup in `histogram()` for grouped non-overlay direct calls.
- [x] Add a focused histogram-level unittest showing grouped-separate mode ignores irrelevant `multiple`.

### Task 21. Template Validation Convention Audit and Alignment
Location: src/spac/templates/histogram_template.py, src/spac/templates/template_utils.py, src/spac/visualization.py

Status: Done

Implementation remarks:
- Current template behavior mixes string-token normalization (`"None"`, `"auto"`) and native JSON types (bool/int/float).
- For this PR, lock a single explicit convention for histogram template parameters:
  - token fields are token-normalized,
  - typed fields remain typed-first,
  - avoid broad “string-only” coercion.
- `Max_Groups` must follow explicit token contract and be consistent between template and core.
- Broader repo-wide template convention cleanup is out of scope for this PR; the task closes once histogram-specific introduced behavior is audited and aligned.

Action items:
- [x] Audit histogram-template parameters and classify each as token-field vs typed-field.
- [x] Normalize token-fields at template boundary (including `Group_by` and `Max_Groups`) using explicit token rules for the current task scope.
- [x] Keep typed fields as typed-first and avoid adding broad `"False"/"True"/"1"` string emulation.
- [x] Finalize current `Max_Groups` template contract for this PR: positive integer or `"unlimited"`; missing/`None`/`"None"` resolve to default `20`.
- [x] Align `histogram()` side parsing with the same effective contract and update focused tests accordingly.
- [x] Review `_parse_optional_number` in `histogram()` and lock one approach: keep with narrowed scope, simplify usage, or replace with explicit local parsing for current hint contracts.
