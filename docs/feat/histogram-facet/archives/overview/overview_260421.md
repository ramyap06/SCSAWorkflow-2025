# Histogram Facet Refactoring Overview

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
  - [Task Details](./task-details.md)
  - [Decisions](./decisions.md)
  - [Implementation Log](./implementation-log.md)

---

## Immediate Next Step

None currently.

## Progress

### Ongoing Tasks
None currently.

### Remaining Tasks
None currently.

### Addressed Tasks
18. Facet Histogram Download DataFrame Contract.
19. Numeric-Annotation Facet Smoke Coverage.
7. Visualization Histogram Unit-Test Completion.
20. Plotting-Control Validation Simplification.
11. Facet Long X-Label Layout Handling.
17. Grouped `group_by` Max-Group Guardrail Validation.
16. Facet Figure Title Layout Handling.
21. Template Validation Convention Audit and Alignment.
15. Facet Layout Hint Validation Simplification.
14. Core Input Normalization Refactor.
13. Facet Geometry Helper API/Docstring + Independent Unittests.
2. Histogram/Template Parameter Boundary.
3. Grouped Annotation Title Bugfix.
6. Facet Label Strategy and Test Alignment.
8. Template Histogram Unit-Test Completion.
9. Facet `ax` Guardrail and Figure Lifecycle Refactor.
12. Facet Plot Test Decomposition and Smoke-Path Contract.
10. Helper Boundary Relocation and Naming Alignment (Facet Scope).
1. Shared Global Bins Helper.
5. Bins Default-Like Fallback Policy and Behavior.
4. Facet Layout Derivation and Non-Facet Kwarg Guardrails.

### Issues (Open)
None currently.

---

## Analysis Summary

### Current Code Structure (Latest)
1. Histogram core and helper flow are in src/spac/visualization.py:
   - `histogram` parses/validates facet layout hints explicitly in its grouped facet path.
   - `_derive_facet_geometry` now assumes pre-normalized inputs and focuses on geometry derivation plus long-label default sizing heuristics.
   - histogram-local helpers `build_grouped_histogram_table` and `resolve_hist_axis_labels` are used across grouped/facet/single histogram paths.
   - Histogram uses figure-level labels in facet mode (clears per-axis labels, sets supxlabel/supylabel).
   - Bins default-like fallback logic now applies the in-house Rice rule only for numeric data; categorical default-like bins stay non-computational.
2. Template wrapper in src/spac/templates/histogram_template.py:
   - Facet and facet_ncol are exposed as user controls.
   - Figure size contract remains at template layer and is passed internally as `facet_fig_width` / `facet_fig_height` hints.
   - `Figure_Width` / `Figure_Height` may remain `"auto"` in facet mode so core geometry can size the figure automatically.
   - Template keeps bins policy and layout-hint normalization at the boundary, while seaborn-native plotting controls are forwarded with lighter validation.
3. Current focused test state (tests/test_visualization/test_histogram.py):
   - Current focused verification remains green for targeted histogram/template checks after grouped-bin cleanup and test compression (`52 passed, 1 warning`).
   - Task 11 implementation is now present in code and covered by focused histogram regressions.

### Codebase Pattern Findings (Concise)
1. Visualization-specific shared logic should stay in visualization.py module-level helpers.
2. Template layer owns user-facing parameter normalization and UX contract.
3. Bare plotting functions should focus on plotting semantics and internal consistency.

---

## Development Details
See [task-details.md](./task-details.md) for the full task-by-task record, implementation remarks, and checked action items.

## Decision Log
See [decisions.md](./decisions.md) for the full decision history and rationale record.

## Execution Log
See [implementation-log.md](./implementation-log.md) for the full dated implementation and verification history.

---

## Future Work (Out of Scope for This PR)

- Potential support for externally provided axes in facet mode (`facet=True` + external `ax`).
- If implemented later, define explicit ownership rules for figure/axes lifecycle and provide dedicated API/tests.
- Possible simplification of facet plotting internals.
- Potential refinement of facet geometry model to account for suptitle/supylabel layout effects (defer to future developers).
- Remaining non-facet histogram return-data consistency follow-up (out of this PR): unify `hist_data` contract across non-facet histogram branches now that Task 18 has narrowed the facet download contract.
- Raw-data / KDE fidelity versus count-table output contract across histogram branches.
- Revisit whether histogram-local helpers with cross-visualization value should move into `spac.utils` in a later cleanup PR.
- Consider a small internal shared coercion helper only if facet-hint parsing expands enough to justify more abstraction again.
- Histogram template parameter-name consistency cleanup across blueprint/template/test payloads (defer; not in facet PR scope).
- Possible double-checking of binning logic.
- UI axis abbreviation follow-up for a later PR only; keep it out of Task 11.

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
