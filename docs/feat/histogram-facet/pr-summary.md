## Related PR

This PR depends on PR #328.

## Summary

This PR finalizes histogram facet support in `SCSAWorkflow` by adding a
faceted grouped-histogram path in `spac.visualization.histogram()` and
aligning `histogram_template.py` with the same facet/grouped contract.

## Changes

- Adds `facet=True` grouped histogram support with shared bins and shared
  category slots, adaptive facet geometry, figure-level labels, grouped
  count-table returns, and grouped-mode guardrails.
- Aligns template-side validation and forwarding for `Facet`, `Facet_Ncol`,
  `Max_Groups`, facet size hints, and overlay-only `multiple`, while keeping
  grouped-only and facet-only hints inactive outside their modes.
- Adds focused tests for `_derive_facet_geometry()`, facet behavior, bins
  fallback, shared-bin consistency, `max_groups`, and external-`ax`
  guardrails.

## Testing

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
- `54 passed`

## Notes For Review

- Suggested review order:
  1. `src/spac/visualization.py` for the facet plotting path, shared-bin
     behavior, and grouped return-data contract.
  2. `src/spac/templates/histogram_template.py` for template-side validation,
     forwarding, and layout handling.
  3. `tests/test_visualization/test_histogram.py`,
     `tests/test_visualization/test_derive_facet_geometry.py`, and
     `tests/templates/test_histogram_template.py` for focused coverage.
- More implementation details, direct diff links, and figure references are
  in [pr-summary-details.md](./pr-summary-details.md).
