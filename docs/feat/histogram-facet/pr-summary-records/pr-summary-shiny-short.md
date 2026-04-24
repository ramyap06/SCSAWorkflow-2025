## Description

This PR finalizes histogram facet support in `SCSAWorkflow` by aligning
`spac.visualization.histogram()` and `histogram_template.py` around one
facet/grouped contract.

**This PR adds faceted grouped histograms with adaptive layout, shared bins,
grouped count-table returns, and clearer grouped/facet guardrails.**

## Related Issue/PR

This PR depends on PR #328.

## Changes

- Core histogram updates:
  - adds `facet=True` grouped plotting
  - adds adaptive facet geometry for group count, size hints, and long labels
  - uses shared bins/shared category slots across grouped facets
  - returns grouped histogram-bin counts instead of raw facet plot input
  - applies figure-level labels and blocks unsupported external-`ax` cases
- Template updates:
  - validates and forwards `Facet`, `Facet_Ncol`, `Max_Groups`, and facet size
    hints
  - keeps grouped-only and facet-only hints inactive outside their modes
  - forwards `multiple` only for grouped same-axis overlays
  - keeps facet title/layout handling readable on denser grids
- Tests and docs:
  - adds dedicated `_derive_facet_geometry()` coverage
  - expands histogram tests for facet behavior, bins fallback, shared-bin
    consistency, `max_groups`, and external-`ax` guardrails
  - updates histogram/template docstrings to match the final contract

## Verification Results

- [x] Focused histogram/helper/template tests passing
  - `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
  - `54 passed`
- [x] Facet mode returns multi-axis output with figure-level labels
- [x] Shared bin alignment verified for numeric and categorical grouped facets
- [x] Non-facet and non-grouped paths ignore irrelevant grouped/facet hints

**Ready for Review**
