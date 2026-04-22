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
  - [Task Details](./task-details.md)
  - [Decisions](./decisions.md)
  - [Implementation Log](./implementation-log.md)
  - [Code Review 2026-04-21](./code-review-2026-04-21.md)

---

## Immediate Next Step

Issue 1. Fix template `Figure_Width` / `Figure_Height` zero-value validation, then Issue 2. Stop forwarding irrelevant template `Max_Groups` when `Group_by` is unset.

## Progress

### Ongoing Tasks
None currently.

### Remaining Tasks
None currently.

### Addressed Tasks
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

### Issues (Open)
1. Template `Figure_Width` / `Figure_Height` validation currently uses truthiness, so an explicit zero can bypass the positive-value check and silently fall back to the default figure size instead of raising.
2. Template `Max_Groups` is still forwarded when `Group_by` is unset, so an ungrouped template call can fail on an irrelevant `Max_Groups` value instead of ignoring it.

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

### Issues

- Fix bugs in `together=False`, `facet=False` branch:
    - Incorrect output `hist_data` (currently repeatedly rewrote during the loop);
    - Overlapping label issues for long labels -> may reuse or imitate the facet geometry derivation logic.
- Naming issues with template parameters in JSON (not consistent with blueprint, e.g. `"Table_"`). May also need to update the blueprint and galaxy-related files with the newly introduced APIs.

### Possible Enhancement (Need Evaluation)

- Confirm with George whether histogram template tests should remain I/O-oriented only or expand to handled-validation coverage.
- UI follow-up for long axis labels: allow abbreviation of labels; label-level fontsize setting.
- Output plot-related data in addition to the existing hist_data dataframe. e.g. add another column of the actual `stat` (e.g. `frequency`) in addition to the `count`.
- `kwargs` expansion:
    - Allow more seaborn `kwargs`;
    - Allow more values for existing `kwargs`;
    - A special case is `KDE`: this requires raw data plotting rather than pre-computed hist data by `calculate_histogram` function.
- External-`ax` support for group-separate plotting (`group_by` is not `None`, `together=False`), including for facet mode.

### Possible Refactor (Need Evaluation)

- Refactor/simplify helper functions inside `histogram` function, and decide whether to relocate to module-level or `utils` folder (with unittests).
- Double-check facet geometry derivation flow in histogram function. Current derivation uses a complex algorithm.
- Double-check layout settings for facet mode in histogram template. Current algorithm uses magic numbers to solve overlapping between titles and subplots.

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
