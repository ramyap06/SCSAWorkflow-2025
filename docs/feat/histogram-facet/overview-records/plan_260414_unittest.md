# Histogram Facet Refactoring Plan (Updated)

## Problem Statement
The histogram facet feature in src/spac/visualization.py is implemented and close to review, but several concerns still need explicit closure.

### Addressed Concerns
1. Duplicated global_bin_edges computation in together and facet paths.
2. Parameter boundary between template and bare histogram function.
3. Grouped annotation title bug in non-facet branch.
4. Facet layout derivation and guardrails.
5. Facet-only kwargs leakage into seaborn non-facet paths.
6. Bins default-like handling performance concern for missing/None/auto-like values.

### Remaining Concerns
1. Test inconsistency: facet test still checks per-axis labels while implementation uses figure-level labels.
2. Missing regression tests for bins fallback normalization and facet bin consistency.
3. Missing template tests for new facet/bins/validation behavior in histogram_template.
4. Test warning: many open figures during test run.
5. Need a compact test strategy decision: keep case-oriented tests vs split focused tests for maintainability.

### To-Decide (Before Implementation)
1. Scope: whether this cycle tests only facet-related contributions or also cross-mode consistency contracts.
2. Facet + external ax behavior: explicitly ban with validation error vs add support.
3. Figure lifecycle policy: whether to refine open/close behavior now or defer with test cleanup only.
4. Facet plotting internals: keep current FacetGrid + map_dataframe + tick_params behavior as-is vs simplify.
5. Granularity of `test_facet_plot`: keep as one case-oriented test vs split into structure/title/label tests.
6. Bar-level assertions in facet tests: required or unnecessary over-constraint.
7. Helper function boundaries:
   - keep `_parse_histogram_layout_kwargs` / `_derive_facet_geometry` / `_resolve_histogram_axis_labels` / `_compute_global_bin_edges` as module helpers,
   - or move/merge/rename some helpers.
8. Helper test strategy: direct unit tests per helper vs public-function behavioral coverage only.
9. Template test philosophy: keep mostly I/O contract tests vs add minimal value/validation assertions for new params.
10. Coverage for newly introduced kwargs (`shrink`, `alpha`): explicit tests now vs defer.

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
   - 20 passed, 1 failed.
   - Failing test is test_facet_plot due to per-axis label expectations.

### Codebase Pattern Findings (Concise)
1. Visualization-specific shared logic should stay in visualization.py module-level helpers.
2. Template layer owns user-facing parameter normalization and UX contract.
3. Bare plotting functions should focus on plotting semantics and internal consistency.

---

## Development Details

### Task 1. Shared Global Bins Helper
Location: src/spac/visualization.py

Status: Done

Action items:
- [x] Create _compute_global_bin_edges as module-level helper.
- [x] Replace together-mode duplicated bin-edge logic with helper call.
- [x] Replace facet-mode duplicated bin-edge logic with helper call.
- [x] Keep helper documented with concise behavior notes.

### Task 2. Facet API Boundary and Layout Derivation
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py

Status: Mostly done, one documentation follow-up optional

Action items:
- [x] Expose facet and facet_ncol at template layer.
- [x] Pass target_fig_width and target_fig_height as internal hints.
- [x] Keep facet_vertical_threshold/facet_height/facet_aspect out of public API contract.
- [x] Derive facet geometry from figure target size plus grid shape.
- [x] Add panel-size/aspect guardrails.
- [ ] Document facet geometry precedence and formula in template doc/comments.

### Task 3. Grouped Annotation Title Bugfix
Location: src/spac/visualization.py

Status: Done

Action items:
- [x] Restore per-group annotation title assignment in non-facet grouped mode.
- [x] Verify grouped annotation titles render correctly.

### Task 4. Facet Label Strategy and Test Alignment
Location: src/spac/visualization.py, tests/test_visualization/test_histogram.py

Status: Implementation done, tests pending

Implementation plan:
1. Update `test_facet_plot` to keep structure checks (axes collection, facet count, facet titles).
2. Replace per-axis xlabel/ylabel assertions with figure-level label assertions:
   - assert per-axis labels are empty in facet mode,
   - assert `fig._supxlabel` and `fig._supylabel` are present and match expected text.
3. Add one focused regression assertion for label strategy under non-default stat (e.g., `stat='density'`) to confirm y super-label reflects stat mapping.
4. Run only updated tests first for fast feedback, then run full histogram test file.
5. If needed, do minimal assertion tuning to avoid over-constraining matplotlib internals.

Action items:
- [x] Lock decision: facet mode uses figure-level labels.
- [x] Keep per-axis labels empty in facet mode and set supxlabel/supylabel.
- [ ] Update test_facet_plot to assert figure-level labels.
- [ ] Add one regression assertion to lock label strategy.

### Task 5. Bins Policy and Default-Like Fallback Behavior
Location: src/spac/templates/histogram_template.py, src/spac/visualization.py

Status: Policy decided and behavior implemented; tests pending

Action items:
- [x] Keep template as user-facing bins policy entrypoint.
- [x] Keep explicit user bins unchanged.
- [x] Normalize bare histogram default-like bins inputs to Rice-rule fallback (missing/None/auto-like).
- [x] Keep facet numeric shared-bin behavior for cross-facet consistency.
- [ ] Add focused regression tests for bins fallback behavior.

### Task 6. Test and Cleanup Pass
Location: tests/test_visualization/test_histogram.py, workspace/plans/feat/histogram-facet/plan.md, tests/templates/test_histogram_template.py

Status: In progress

Action items:
- [ ] Run focused histogram test file and make it fully green.
- [ ] Add tests for global bin-edge consistency across facets.
- [ ] Add tests for facet layout parameters.
- [ ] Consider axes simplification test if the simplification is implemented.
- [ ] Reduce figure-open warning noise in tests (close figures or fixture cleanup).
- [ ] Keep future UI axis abbreviation task tracked only (no implementation in this PR).

### Task 7. Template Unit Tests for Histogram Changes
Location: tests/templates/test_histogram_template.py, src/spac/templates/histogram_template.py

Status: Not started

Action items:
- [ ] Add facet-mode template test case(s) covering `Facet=True` and `Facet_Ncol` wiring through `run_from_json`.
- [ ] Add template-level bins behavior test case(s) for default-like input (`"auto"`/None path) and explicit positive integer path.
- [ ] Add validation test case(s) for invalid `Facet_Ncol` and invalid plotting enums (`multiple`, `element`, `stat`).
- [ ] Verify output contract remains stable (`saved_files` keys and generated artifacts).
- [ ] Run template test file and keep it green with histogram test updates.

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

### D3. Kwarg Leakage Guardrail (Task 2)
Decision:
Facet layout kwargs must not leak into non-facet seaborn calls.

Details:
Layout hints are parsed and removed before histplot paths.

Rationale:
Prevent accidental runtime errors and behavior drift.

### D4. Label Strategy (Task 4)
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
Keep _compute_global_bin_edges as module-level helper in src/spac/visualization.py.

Details:
Used by multiple grouped/facet flows and suitable for reuse by future visualizations.

Rationale:
Improves reuse and reduces duplicate logic.

---

## Implementation Checklist

### Task 1. Shared Global Bins Helper
Priority: High
- [x] Create _compute_global_bin_edges as module-level helper.
- [x] Replace together-mode duplicated bin-edge logic with helper call.
- [x] Replace facet-mode duplicated bin-edge logic with helper call.
- [x] Keep helper documented with concise behavior notes.

### Task 2. Facet API Boundary and Layout Derivation
Priority: Medium
- [x] Expose facet and facet_ncol at template layer.
- [x] Pass target_fig_width and target_fig_height as internal hints.
- [x] Keep facet_vertical_threshold/facet_height/facet_aspect out of public API contract.
- [x] Derive facet geometry from figure target size plus grid shape.
- [x] Add panel-size/aspect guardrails.
- [ ] Document facet geometry precedence and formula in template doc/comments.

### Task 3. Grouped Annotation Title Bugfix
Priority: High
- [x] Restore per-group annotation title assignment in non-facet grouped mode.
- [x] Verify grouped annotation titles render correctly.

### Task 4. Facet Label Strategy and Test Alignment
Priority: Medium
- [x] Lock decision: facet mode uses figure-level labels.
- [x] Keep per-axis labels empty in facet mode and set supxlabel/supylabel.
- [ ] Update test_facet_plot expectations to figure-level labels.
- [ ] Add one regression assertion to lock label strategy.

### Task 5. Bins Policy and Default-Like Fallback Behavior
Priority: Medium
- [x] Keep template as user-facing bins policy entrypoint.
- [x] Keep explicit user bins unchanged.
- [x] Normalize bare histogram default-like bins inputs to Rice-rule fallback (missing/None/auto-like).
- [x] Keep facet numeric shared-bin behavior for cross-facet consistency.
- [ ] Add focused regression tests for bins fallback behavior.

### Task 6. Test and Cleanup Pass
Priority: Medium
- [ ] Run focused histogram test file and make it fully green.
- [ ] Add tests for global bin-edge consistency across facets.
- [ ] Add tests for facet layout parameters.
- [ ] Consider axes simplification test if the simplification is implemented.
- [ ] Reduce figure-open warning noise in tests (close figures or fixture cleanup).
- [ ] Keep future UI axis abbreviation task tracked only (no implementation in this PR).

### Task 7. Template Unit Tests for Histogram Changes
Priority: Medium
- [ ] Add facet-mode template test case(s) for `Facet`/`Facet_Ncol` wiring.
- [ ] Add template-level bins behavior test case(s) for default-like and explicit bins.
- [ ] Add validation test case(s) for invalid `Facet_Ncol` and plotting enums.
- [ ] Verify template output contract remains stable.
- [ ] Run tests/templates/test_histogram_template.py successfully.

---

## Notes for Implementation

- Run tests after each change: python -m pytest tests/test_visualization/test_histogram.py
- Current focused status: 20 passed, 1 failed (test_facet_plot label expectation mismatch)
- Keep this PR focused on histogram facet correctness, tests, and contract clarity

---

## Immediate Next Step

Execute Task 4 first, then Task 7, then Task 6:
1. Update test_facet_plot label assertions to figure-level behavior and add the Task 4 regression assertion.
2. Add/adjust template unittests for histogram_template facet/bins/validation coverage (Task 7).
3. Re-run focused histogram and template tests, then finish cleanup (Task 6).
