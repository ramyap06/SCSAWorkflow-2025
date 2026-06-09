# Histogram Facet Development Overview

## Problem Statement

This plan now serves as the completed implementation record for histogram facet development, along with the remaining future-work notes that were intentionally kept out of the PR.

## Project Context

- Environment: `conda activate spac`
- Feature branch: `feat/histogram-facet`
- Target branch: `upstream/dev`
- Scope: histogram facet development across core plotting, template wiring, and focused tests
- Primary files:
  - `src/spac/visualization.py`
  - `src/spac/templates/histogram_template.py`
  - `tests/test_visualization/test_histogram.py`
  - `tests/test_visualization/test_derive_facet_geometry.py`
  - `tests/templates/test_histogram_template.py`
- Detailed records:
  - [Task Details](./development-details/task-details.md)
  - [Decisions](./development-details/decisions.md)
  - [Implementation Log](./development-details/implementation-log.md)
  - [Code Review](./development-details/code-review.md)
  - [PR Summary](./pr-summary.md)
  - [PR Report](./pr-report.md)
  - [PR Details](./pr-summary-details.md)
  - [Future Work](./future-work.md)

---

## Immediate Next Step

Follow up on the remaining open review questions in Issues.

## Progress

### Ongoing Tasks
None currently.

### Remaining Tasks
None currently.

### Postponed Tasks
None currently.

### Dropped Tasks
CR.3. Formatting cleanup for review readiness.

### Addressed Tasks
CR.10. Histogram Parameter Contract Simplification.
CR.8. Move Facet Geometry Helper to Shared Utils.
CR.9. Hard-Code Default Bins Test Expectation.
CR.4. Dedicated PR summary for histogram facet changes.
CR.7. Histogram visualization documentation consistency cleanup.
CR.6. Conditional forwarding of grouped-only and facet-only template hints.
CR.5. Template zero-value figure-size validation.
CR.2. Ignore facet-only size hints when `facet=False`.
CR.1. Documentation/review alignment for `max_groups` and `facet_ncol` direct-call edge cases.
1. Shared Global Bins Helper.
2. Histogram/Template Parameter Boundary.
3. Grouped Annotation Title Bugfix.
4. Facet Layout Derivation and Non-Facet Kwarg Guardrails.
5. Bins Default-Like Fallback Policy and Behavior.
6. Facet Label Strategy and Test Alignment.
7. Visualization Histogram Unit-Test Completion.
8. Template Histogram Unit-Test Completion.
9. Facet `ax` Guardrail and Figure Lifecycle Refactor.
10. Helper Boundary Relocation and Naming Alignment (Facet Scope).
11. Facet Long X-Label Layout Handling.
12. Facet Plot Test Decomposition and Smoke-Path Contract.
13. Facet Geometry Helper API/Docstring + Independent Unittests.
14. Core Input Normalization Refactor.
15. Facet Layout Hint Validation Simplification.
16. Facet Figure Title Layout Handling.
17. Grouped `group_by` Max-Group Guardrail Validation.
18. Facet Histogram Download DataFrame Contract.
19. Numeric-Annotation Facet Smoke Coverage.
20. Plotting-Control Validation Simplification.
21. Template Validation Convention Audit and Alignment.
22. Unused Import Cleanup in Touched Histogram Modules.

### Issues (Open)
1. Follow up on George's question about whether `shrink`, `bins`, `alpha`, and `stat` were removed from the histogram template contract; they were not removed, and this has been clarified in reply while waiting for George's response.
2. Decide whether the histogram template needs additional meaningful tests for template-owned behavior, especially title/suptitle handling, while avoiding broad validation-only tests that only duplicate core or `text_to_value` coverage. (some others include but not limited to: final figure/output handling, and conditional forwarding)
3. Add future grouped-plot behavior for excessive groups: when group count exceeds `max_groups`, plot the most frequent `max_groups` groups with a warning instead of rejecting the plot.

---

## Analysis Summary

### Current Code Structure (Latest)
1. Histogram core and helper flow are in src/spac/visualization.py:
   - `histogram` now expects normalized direct-call values for grouped/facet layout hints and keeps only minimal structural checks at the core boundary.
   - `derive_facet_geometry` lives in `src/spac/utils.py`, assumes pre-normalized inputs, and focuses on geometry derivation plus long-label default sizing heuristics.
   - histogram-local helpers `build_grouped_histogram_table` and `resolve_hist_axis_labels` are used across grouped/facet/single histogram paths.
   - Histogram uses figure-level labels in facet mode (clears per-axis labels, sets supxlabel/supylabel).
   - Core `bins` fallback now treats omitted/`None` and `"auto"` as default-like; loose text aliases are handled at the template boundary rather than direct-call core.
2. Template wrapper in src/spac/templates/histogram_template.py:
   - Facet and facet_ncol are exposed as user controls.
   - Figure size contract remains at template layer and is passed internally as `facet_fig_width` / `facet_fig_height` hints.
   - `Figure_Width` / `Figure_Height` may remain `"auto"` in facet mode so core geometry can size the figure automatically.
   - Template keeps bins policy and layout-hint normalization at the boundary, forwards `multiple` only for grouped same-axis overlays, and ignores grouped/facet-only hints when their modes are inactive.
3. Current focused test state (tests/test_visualization/test_histogram.py):
   - Focused histogram/template verification is green for the current staged CR.10 snapshot (`45 passed, 1 warning`) across `tests/test_visualization/test_histogram.py` and `tests/templates/test_histogram_template.py`.
   - The latest follow-up removed stale direct-call tests for removed core contracts and tightened matching test docstrings after reverting a noisy default-`max_groups` warning.

### Codebase Pattern Findings (Concise)
1. Visualization-specific shared logic should stay in visualization.py module-level helpers.
2. Template layer owns user-facing parameter normalization and UX contract.
3. Bare plotting functions should focus on plotting semantics and internal consistency.

---

## Development Details
See [task-details.md](./development-details/task-details.md) for the full task-by-task record, implementation remarks, and checked action items.

## Decision Log
See [decisions.md](./development-details/decisions.md) for the full decision history and rationale record.

## Execution Log
See [implementation-log.md](./development-details/implementation-log.md) for the full dated implementation and verification history.

---

## Future Work (Out of Scope for This PR)

### Immediate Next Steps

- Follow-up for the naming inconsistency between template and tests (email George)
- Remove the deprecated plotting mode for together=False, facet=False
- Allow auto-filter to most frequent groups with notification (rather than rejecting)
- Do not reject `ax` directly. Provide it in core function (for facet we can just provide our plotting)
- Add unittests for additional functionality on template

### More Ideas

- Allow abbreviation of labels, label-level fontsize setting (current examples are in Shiny side, `feat_vs_anno` tab (hierachical heatmap)) for long-label issues
- More plot-related data in output dataframe, e.g. frequency/proportion if specified in `stat`
- `kwargs` expansion:
  - Allow more seaborn `kwargs`;
  - Allow more values for existing `kwargs`;
  - A special case is `KDE`: this requires raw data plotting rather than pre-computed hist data by `calculate_histogram` function.
- Possible simplification/reloation for helper functions (need evaluation)
- Possible simplification for facet geometry/layout derivation (need evaluation). Current way is driven by AI

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
   - Keep facet test ordering consistent for readability: place smoke/core behavior tests first, then layout-hint contract tests, then deeper consistency/regression checks.
   - Keep meaningful label assertions, but avoid redundant or overly brittle label checks.
   - Use direct helper tests only where helper branching is non-trivial; otherwise cover behavior through public histogram tests.
   - Keep template tests I/O-oriented.
   - For UI-motivated geometry heuristics, prefer relational assertions (for example, larger than default or explicit hints remain authoritative) over exact formula-locked numbers.
   - Hard code the values in the tests.
- Helper-location decision rule (first-principles):
   - Keep a helper at module level only when it has clear cross-function/cross-plot reuse in current or near-term planned work.
   - Keep or move a helper into function scope when logic is specific to one plotting function and exposing it would only increase maintenance surface.
   - Prefer minimal refactor scope for this PR: facet-mode correctness first, broad cleanup later.
- Facet hint validation pattern:
   - Keep template strict for user-facing JSON parameter validation.
   - Keep `histogram` minimally defensive for direct calls.
   - Prefer explicit local parsing over generic normalization helpers when the contract is small and plot-specific.
- Run tests after each change: python -m pytest tests/test_visualization/test_histogram.py
- Keep this PR focused on histogram facet correctness, tests, and contract clarity
