# Histogram Facet Refactoring Plan (Updated)

## Problem Statement

The histogram facet feature in src/spac/visualization.py is implemented and close to review, but several tasks still need explicit closure.

## Immediate Next Step

Continue Task 8/7/5/10/1/4:
1. Execute Task 8 template I/O-only tests.
2. Continue Task 7 visualization coverage, then complete Task 5 default-like bins tests, Task 10 helper relocation updates, Task 1 shared-bin numerical/categorical tests, and Task 4 facet-layout parameter tests.

## Progress

### Ongoing Tasks
1. Shared Global Bins Helper.
4. Facet Layout Derivation and Non-Facet Kwarg Guardrails.
5. Bins Default-Like Fallback Policy and Behavior.

### Remaining Tasks
7. Visualization Histogram Unit-Test Completion.
8. Template Histogram Unit-Test Completion.
10. Helper Boundary Relocation and Naming Alignment (Facet Scope).
11. Facet Long X-Label Layout Handling.

### Addressed Tasks
2. Histogram/Template Parameter Boundary.
3. Grouped Annotation Title Bugfix.
6. Facet Label Strategy and Test Alignment.
9. Facet `ax` Guardrail and Figure Lifecycle Refactor.
12. Facet Plot Test Decomposition and Smoke-Path Contract.

### Issues (Open)
None currently.

---

## Analysis Summary

### Current Code Structure (Latest)
1. Histogram core and helper flow are in src/spac/visualization.py:
   - _compute_global_bin_edges shared helper exists and is used in both together and facet grouped flows.
   - _parse_histogram_layout_kwargs and _derive_facet_geometry enforce internal facet layout handling.
   - Histogram uses figure-level labels in facet mode (clears per-axis labels, sets supxlabel/supylabel).
   - Bins default-like fallback logic now normalizes missing/None/auto-like inputs to in-house Rice rule in bare histogram path.
2. Template wrapper in src/spac/templates/histogram_template.py:
   - Facet and facet_ncol are exposed as user controls.
   - Figure size contract remains at template layer and is passed internally as target_fig_width/target_fig_height.
   - Template keeps validation for plotting controls and bins policy entrypoint.
3. Current focused test state (tests/test_visualization/test_histogram.py):
   - 24 passed, 0 failed.
   - Facet decomposition and non-default stat label regression assertion are completed.

### Codebase Pattern Findings (Concise)
1. Visualization-specific shared logic should stay in visualization.py module-level helpers.
2. Template layer owns user-facing parameter normalization and UX contract.
3. Bare plotting functions should focus on plotting semantics and internal consistency.

---

## Development Details

### Task 1. Shared Global Bins Helper
Location: src/spac/visualization.py

Status: Done, tests pending

Action items:
- [x] Create _compute_global_bin_edges as module-level helper.
- [x] Replace together-mode duplicated bin-edge logic with helper call.
- [x] Replace facet-mode duplicated bin-edge logic with helper call.
- [x] Keep helper documented with concise behavior notes.
- [ ] Add shared-bin regression tests here, explicitly covering both numerical and categorical input cases.

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

Status: Mostly done, one documentation follow-up optional

Action items:
- [x] Derive facet geometry from figure target size plus grid shape.
- [x] Add panel-size/aspect guardrails.
- [x] Facet layout kwargs must not leak into non-facet seaborn calls.
- [x] Lock decision for this PR: keep current `_derive_facet_geometry` ratio approximation.
- [ ] Document facet geometry precedence and formula in template doc/comments.
- [ ] Add unit tests for facet layout parameters, including `facet_ncol` valid/invalid/type and `facet_fig_width`/`facet_fig_height` checks.

### Task 5. Bins Default-Like Fallback Policy and Behavior
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py

Status: Policy decided and behavior implemented; tests pending

Action items:
- [x] Keep template as user-facing bins policy entrypoint.
- [x] Keep explicit user bins unchanged.
- [x] Normalize bare histogram default-like bins inputs to Rice-rule fallback (missing/None/auto-like).
- [x] Keep facet numeric shared-bin behavior for cross-facet consistency.
- [ ] Add focused regression tests for default-like bins fallback behavior.

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

Status: In progress

Action items:
- [x] Run focused histogram test file and make it fully green.
- [ ] Complete remaining histogram coverage after Task 12 decomposition/smoke-path work.
- [ ] Add/adjust facet validation unit tests (`facet=True` requires `group_by`, and `facet` with `together=True` conflict).
- [ ] Add unit tests for newly introduced kwargs (`shrink`, `alpha`).
- [ ] Add one focused facet regression test for annotation-based categorical data using lightweight assertions.
- [ ] Add additional missed unit-test cases identified in the final gap scan, and verify all introduced facet logic paths are covered.

### Task 8. Template Histogram Unit-Test Completion
Location: tests/templates/test_histogram_template.py

Status: Not started

Action items:
- [ ] Verify template output contract remains stable (`saved_files` keys and generated artifacts).
- [ ] Remove duplicate facet x-label reassignment in template: keep label content from `histogram`; template only applies presentation adjustments (e.g., rotation) if needed.
- [ ] Run template test file and verify passing status with histogram test updates.

### Task 9. Facet `ax` Guardrail and Figure Lifecycle Refactor
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Reject `facet=True` when external `ax` is provided, with a clear validation message.
- [x] Refactor histogram figure creation/closure flow to avoid throwaway figures and keep strict internal-ownership closing.
- [x] Add minimal regression tests for the guardrail and lifecycle behavior.

### Task 10. Helper Boundary Relocation and Naming Alignment (Facet Scope)
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Not started

Action items:
- [ ] Move `compute_global_bin_edges` and `resolve_hist_axis_labels` logic into `histogram` scope.
- [ ] Add/adjust histogram-level tests to match relocated helper boundaries under CONTRIBUTING.md expectations.

### Task 11. Facet Long X-Label Layout Handling
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py, tests/test_visualization/test_histogram.py, tests/templates/test_histogram_template.py

Status: Not started

Action items:
- [ ] Add a targeted facet-mode layout adjustment so long/rotated x-label text does not get hidden or overlap neighboring facets.
- [ ] Document tradeoffs against strict facet geometry proportionality for the selected layout adjustment.
- [ ] Add focused regression coverage for long-label facet readability behavior (annotation/categorical case included).

### Task 12. Facet Plot Test Decomposition and Smoke-Path Contract
Location: tests/test_visualization/test_histogram.py

Status: Done

Action items:
- [x] Split the current heavy facet test into focused tests for structure, title mapping, label policy, and rendering smoke-path checks.
- [x] Add one thin facet smoke-path test that verifies end-to-end execution and basic plotted output presence, including lightweight bar-level presence checks (non-empty facet patches/artists).
- [x] Add split assertions that validate facet behavior using stable, non-geometry-sensitive checks.
- [x] Run focused facet tests first, then run the full histogram test file.

---

## Decision Log

### D1. Facet Parameter Exposure (Task 2)
Decision:
Expose only facet and facet_ncol as user-facing facet controls.

Details:
Figure_Width/Figure_Height/Figure_DPI remain template contract; target_fig_width/target_fig_height stay internal hints.

Rationale:
Keep user API simple and avoid competing layout knobs.

### D2. Sizing Precedence Rule (Task 2)
Decision:
Template figure sizing is authoritative.

Details:
Histogram derives internal facet panel geometry from target figure size and facet grid shape.

Rationale:
Predictable output and cleaner cross-template behavior.

### D3. Kwarg Leakage Guardrail (Task 4)
Decision:
Facet layout kwargs must not leak into non-facet seaborn calls.

Details:
Layout hints are parsed and removed before histplot paths.

Rationale:
Prevent accidental runtime errors and behavior drift.

### D4. Label Strategy (Task 6)
Decision:
Use figure-level labels in facet mode.

Details:
Facet axes clear per-axis labels; figure sets supxlabel/supylabel.

Rationale:
Cleaner multi-panel readability and consistent presentation.

### D5. No Deprecation Cycle Needed (Task 2)
Decision:
No deprecation policy needed for removed facet geometry kwargs in user API.

Details:
Changes are new and not part of long-standing external template contract.

Rationale:
Keep implementation clean without migration overhead.

### D6. Bins Policy Scope (Task 5)
Decision:
Template owns user-facing bins policy; bare histogram keeps internal fallback normalization.

Details:
Do not perform broad legacy refactor; keep targeted default-like bins fallback and facet shared-bin consistency.

Rationale:
Balance stability with performance and consistency.

### D7. Helper Function Placement (Task 1)
Decision:
Adopt mixed helper boundaries for facet scope and keep agreed names.

Details:
- Keep `_parse_facet_layout_hints` and `_derive_facet_geometry` at module level.
- Move `compute_global_bin_edges` and `resolve_hist_axis_labels` into `histogram` scope.
- Keep `cal_bin_num` and `calculate_histogram` nested and unchanged.

Rationale:
Preserve reuse for facet-layout logic while minimizing histogram-only API surface and avoiding unnecessary renaming churn.

### D8. Testing Scope and Granularity (Notes for Implementation, Task TBD)
Decision:
Use a hybrid test strategy: expand existing tests where helpful, split high-friction facet cases, keep a small cross-mode consistency matrix, and keep template tests I/O-oriented.

Details:
- Keep old tests and expand selected assertions when behavior changed.
- Split the current facet test into focused structure/title/label checks if that improves readability.
- Use direct helper tests only for helpers with meaningful branching; otherwise cover via public histogram behavior.
- Keep template tests primarily focused on I/O artifacts and wiring.

Rationale:
This satisfies CONTRIBUTING.md coverage expectations without over-testing internals or duplicating behavior across too many files.

### D9. Coverage Policy for New Facet/Template Tests (Notes for Implementation, Tasks TBD)
Decision:
Write targeted regression tests for the new facet contracts, bins fallback behavior, and template facet parameter wiring.

Details:
- Keep meaningful label assertions.
- Avoid redundant or over-constraining rendering assertions.
- Validate new kwargs and template outputs at the level of the public contract.

Rationale:
Protect the actual user-visible behavior while keeping the suite maintainable.

### D10. Facet `ax` + Lifecycle Refactor Scope (Task 9)
Decision:
For this PR, ban `facet=True` with external `ax`, refactor figure lifecycle handling, and keep existing facet internals unchanged.

Details:
Apply a targeted change only: explicit validation for unsupported external-`ax` facet usage, plus ownership-safe figure creation/closure cleanup.

Rationale:
Addresses notebook/lifecycle issues without broad internal redesign.

### D11. Facet Bin/Bar/Label Test Scope (Task 7)
Decision:
Keep bar-level assertions lightweight across Task 12 split/smoke tests and Task 7 follow-up coverage, place shared-bin coverage under Task 1, and keep default-like bins fallback coverage under Task 5.

Details:
- Cover facet-specific bin contracts directly while avoiding brittle rendering-detail assertions.
- Split coverage by concern: shared-bin contracts in Task 1, default-like bins fallback in Task 5, label contracts in Task 6, and remaining histogram/facet checks in Task 7.

Rationale:
- Protects key facet behavior with stable, maintainable tests.
- Reduces Task 7 scope while keeping ownership of each test group clear.

### D12. Template Test Scope Restriction (Task 8)
Decision:
Template histogram tests remain I/O-focused for this PR.

Details:
Avoid template-side logic tests; keep only output-contract coverage. Non-I/O template assertions are out of scope for this PR.

Rationale:
Aligns with repository template-test style and avoids duplicating visualization logic checks.

### D13. Facet Geometry Ratio Policy (Task 4)
Decision:
Keep current `_derive_facet_geometry` ratio approximation for this PR.

Details:
Do not adjust the geometry model for suptitle/suplabel effects in this PR.

Rationale:
Minimize layout regression risk and keep scope focused on facet correctness and tests.

### D14. Facet Figure Title Ownership (Task 8)
Decision:
Do not add figure-level suptitle in `histogram`; keep figure title responsibility in template/post-processing layer.

Details:
`histogram` keeps plotting semantics and axis/super-label behavior; template controls user-facing figure title composition.

Rationale:
Preserves layer boundaries, keeps facet-mode scope focused, and matches current template-first user-facing workflow.

### D15. Facet X-Label Ownership and Duplication Rule (Task 8)
Decision:
For facet mode, keep x-label semantic content in `histogram`; template may adjust label presentation only.

Details:
Avoid recomputing/reassigning facet x-label text in template when it already comes from `histogram`.

Rationale:
Prevents duplicated labeling logic and reduces drift between plotting core and template post-processing.

### D16. Facet Annotation/Categorical Test Depth Resolution (Task 7)
Decision:
Use a mixture approach: keep lightweight facet bar assertions, and add one focused annotation-based categorical facet regression test.

Details:
Avoid strict bar center/height rendering assertions for facet plots; ensure annotation/categorical facet behavior is covered with stable, contract-level checks.

Rationale:
Closes the annotation/categorical coverage gap while keeping facet tests robust and maintainable.

### D17. Facet Label-Issue Resolution Path (Task 11)
Decision:
Solve the facet long x-label overlap issue through Task 11 implementation work.

Details:
For this issue, the chosen path is implementation via Task 11 (layout handling), while abbreviation exploration remains deferred.

Rationale:
Keeps this refactoring decision concrete and scoped to facet label readability behavior in this PR.

### D18. Facet Test Split Task Boundary (Task 12)
Decision:
Track facet-test decomposition as dedicated Task 12 and remove overlapping split/smoke scope from Task 7.

Details:
- Task 6 owns figure-level label policy checks and non-default stat label mapping.
- Task 12 owns splitting the heavy facet test and defining one smoke-path facet test.
- Task 7 owns remaining histogram/facet coverage beyond the split work.

Rationale:
Clarifies ownership, avoids duplicated action items, and keeps facet-test changes reviewable.

---

## Task Execution Log

- 2026-04-16: Completed Task 9 implementation and focused validation.
   - Added external-`ax` guardrail for unsupported grouped-separate/facet layouts in `histogram`.
   - Refactored figure lifecycle flow to avoid throwaway figure creation/closure in grouped paths.
   - Added regression coverage in `tests/test_visualization/test_histogram.py` for external-`ax` support/rejection modes.
   - Verified focused tests in `spac` environment:
      - `conda run -n spac python -m unittest discover -s tests/test_visualization -p test_histogram.py -k ax_passed_as_argument`
      - `conda run -n spac python -m unittest discover -s tests/test_visualization -p test_histogram.py -k external_ax_guardrail_modes`

- 2026-04-16: Completed Task 6 test-failure fix and histogram test verification.
   - Updated `test_facet_plot` in `tests/test_visualization/test_histogram.py` to assert facet figure-level labels and empty per-axis labels.
   - Verified targeted fix and full histogram test file in `spac` environment:
      - `conda run -n spac pytest tests/test_visualization/test_histogram.py::TestHistogram::test_facet_plot -q`
      - `conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (22 passed)

- 2026-04-16: Completed Task 12 facet-test decomposition and finalized Task 6 stat-mapping coverage.
   - Reorganized facet tests in `tests/test_visualization/test_histogram.py` into scenario-based checks: smoke+structure, titles+label policy, and non-default stat label mapping.
   - Added explicit facet non-default stat regression assertion for `stat='density'` figure-level y-label behavior.
   - Verified updated histogram test file in `spac` environment:
      - `conda run -n spac pytest tests/test_visualization/test_histogram.py -q` (24 passed)

---

## Future Work (Out of Scope for This PR)

- Potential support for externally provided axes in facet mode (`facet=True` + external `ax`).
- If implemented later, define explicit ownership rules for figure/axes lifecycle and provide dedicated API/tests.
- Possible simplification of facet plotting internals.
- Potential refinement of facet geometry model to account for suptitle/supylabel layout effects (defer to future developers).
- Possible double-checking of binning logic.
- UI axis abbreviation follow-up (no implementation in this PR).
- Optional axes simplification exploration and any related tests.

---

## Notes for Implementation

- Follow CONTRIBUTING.md unittest requirements for changed/new code:
   - write tests for all new/updated functions and code paths,
   - include corner cases and handled exceptions,
   - aim for comprehensive coverage with clear ground truth checks.
- Follow the following rules of writing unittests
   - Allow cross-mode consistency tests where they protect shared behavior introduced by facet changes in this PR.
   - Keep old tests and expand selected assertions when needed; do not delete baseline coverage.
   - Prefer splitting high-friction tests into structure/title/label-focused checks, while keeping case-oriented tests when they remain readable.
   - Keep meaningful label assertions, but avoid redundant or overly brittle label checks.
   - Use direct helper tests only where helper branching is non-trivial; otherwise cover behavior through public histogram tests.
   - Keep template tests I/O-oriented.
- Helper-location decision rule (first-principles):
   - Keep a helper at module level only when it has clear cross-function/cross-plot reuse in current or near-term planned work.
   - Keep or move a helper into function scope when logic is specific to one plotting function and exposing it would only increase maintenance surface.
   - Prefer minimal refactor scope for this PR: facet-mode correctness first, broad cleanup later.
- Run tests after each change: python -m pytest tests/test_visualization/test_histogram.py
- Keep this PR focused on histogram facet correctness, tests, and contract clarity

