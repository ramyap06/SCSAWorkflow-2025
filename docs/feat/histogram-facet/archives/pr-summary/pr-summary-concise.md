## Related PR

This PR depends on PR #328.

## Summary

This PR finalizes histogram facet support in `SCSAWorkflow` by adding a
faceted grouped-histogram path in `spac.visualization.histogram()` and
aligning `histogram_template.py` with the same facet/grouped contract.

## Changes

- Adds `facet=True` grouped histogram support with shared bins, adaptive facet
  geometry, figure-level labels, grouped count-table returns, and grouped-mode
  guardrails.
- Aligns template-side validation and forwarding for `Facet`, `Facet_Ncol`,
  `Max_Groups`, facet size hints, and overlay-only `multiple`.
- Adds focused tests for facet behavior, `_derive_facet_geometry()`, bins
  fallback, shared-bin consistency, `max_groups`, and external-`ax`
  guardrails.

## Testing

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
- `54 passed`

## More Details

Additional implementation notes for reviewers are in
[pr-summary-concise-details.md](./pr-summary-concise-details.md).
