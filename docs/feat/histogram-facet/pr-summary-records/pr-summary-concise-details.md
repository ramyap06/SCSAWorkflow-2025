# Detailed PR Notes

## Purpose

This file is the reviewer appendix for the concise PR body in
[pr-summary-concise.md](./pr-summary-concise.md). The goal is to keep the PR
body short while preserving enough implementation detail for reviewers who
want a faster map of the full histogram facet diff.

## Main Change Areas

### 1. Facet Plotting Path in `histogram()`

- Adds `facet=True` as a grouped plotting mode in
  `spac.visualization.histogram()`.
- Requires `group_by` when faceting and rejects `together=True` with
  `facet=True`.
- Uses `_derive_facet_geometry()` to choose facet columns and panel geometry
  from group count, explicit figure-size hints, and long rotated tick labels.
- Applies figure-level x/y labels in facet mode instead of repeating axis
  labels on every panel.
- Rejects unsupported external-`ax` usage for grouped-separate and facet
  layouts.

### 2. Shared Bins and Returned DataFrame Contract

- Reuses one grouped histogram-bin table builder across grouped-overlay and
  facet grouped paths.
- Keeps numeric facet panels aligned by reusing shared bin edges across
  groups.
- Keeps categorical facet panels aligned by preserving shared category slots,
  including zero-count categories within individual groups.
- Standardizes the facet return `df` around grouped histogram-bin counts plus
  grouping metadata instead of returning raw plotting input.

### 3. Bins and Guardrail Behavior

- Default-like `bins` values (`None`, `"auto"`, `"none"`, `""`) fall back to
  the Rice-rule estimator for numeric histograms.
- Categorical grouped/facet paths ignore numeric-style `bins` variation and
  preserve category-slot alignment instead.
- Adds grouped-mode `max_groups` validation with default threshold `20`.
- Supports explicit positive overrides and `"unlimited"` bypass behavior.

### 4. Template Boundary Changes

- `histogram_template.py` now exposes and validates `Facet`, `Facet_Ncol`,
  and `Max_Groups`.
- `Figure_Width` / `Figure_Height` use `"auto"` defaults in facet mode so the
  core geometry logic can determine size when explicit hints are absent.
- Explicit zero figure sizes now fail fast instead of bypassing validation.
- `facet_ncol`, `facet_fig_width`, `facet_fig_height`, and
  `facet_tick_rotation` are forwarded only when `facet=True`.
- `max_groups` is forwarded only when `group_by` is active.
- `multiple` is forwarded only for grouped same-axis overlays.
- `element` and `stat` stay lightly normalized without adding extra SPAC-only
  restrictions.

### 5. Title and Layout Handling

- Facet titles and figure-level title handling were adjusted so denser grids
  stay more readable.
- Long rotated categorical labels feed into default facet geometry heuristics.
- Explicit facet figure-size hints remain authoritative when provided.

## Tests Added or Expanded

- Dedicated tests for `_derive_facet_geometry()`.
- Facet smoke-path and output-structure coverage.
- Figure-level label policy checks.
- Numeric-annotation facet coverage.
- Shared-bin consistency tests for numeric and categorical grouped facets.
- Facet-size-hint validation and paired-hint checks.
- Long-label geometry behavior.
- `max_groups` default, override, unlimited, and invalid-input behavior.
- External-`ax` guardrails.
- Categorical facet `bins` handling.

## Suggested Review Order

1. `src/spac/visualization.py`
   Focus on the facet plotting path, shared-bin handling, returned `df`
   contract, and grouped/facet guardrails.
2. `src/spac/templates/histogram_template.py`
   Focus on user-facing validation, conditional forwarding, and layout/title
   handling.
3. `tests/test_visualization/test_histogram.py`,
   `tests/test_visualization/test_derive_facet_geometry.py`, and
   `tests/templates/test_histogram_template.py`
   Focus on whether the coverage matches the newly introduced contracts.

## Focused Verification

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
- `54 passed`
