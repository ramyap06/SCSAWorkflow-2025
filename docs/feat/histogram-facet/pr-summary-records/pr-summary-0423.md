## Related PR

This PR depends on PR #328.

## Summary

This PR finalizes histogram facet support in `SCSAWorkflow` by adding a faceted grouped-histogram path in `spac.visualization.histogram()` and aligning `histogram_template.py` with the same facet/grouped contract.

## Changes

- Adds `facet=True` grouped histogram support with shared bins, adaptive facet geometry, figure-level labels, grouped count-table returns, and grouped-mode guardrails.
- Aligns template-side validation and forwarding for `Facet`, `Facet_Ncol`, `Max_Groups`, facet size hints, and overlay-only `multiple`.
- Adds focused tests for facet behavior, `_derive_facet_geometry()`, bins fallback, shared-bin consistency, `max_groups`, and external-`ax` guardrails.

## Details

Adds `facet=True` grouped histogram support with shared bins, adaptive facet geometry, figure-level labels, grouped count-table returns, and grouped-mode guardrails.
- A new grouped (`group_by`) plotting mode `facet=True` in `spac.visualization.histogram()`, with `together=False`
- Reuses one grouped histogram-bin table builder `build_grouped_histogram_table()` across grouped-together and facet paths to get
  - Grouped histogram-bin counts return `df` instead of raw data (which was the case in previous PR)
  - Shared bin edges across groups in both numeric and categorical cases
- Adaptive facet geometry computed by a new helper function `spac.visualization._derive_facet_geometry()` based on
  - Group count (`n_groups`)
  - Optional figure-size hints: `facet_fig_width`, `facet_fig_height`
  - Optional layout hints: label length, label rotation (`facet_tick_rotation`)
  - Optional user specified hints: `facet_ncol` and more
- Figure level `x`/`y` labels and figure level title in facet mode instead of axes-level labels and titles 
- More guardrails
  - Rejects external `ax` and `multiple` for grouped-separate and facet paths (`group by` set, `together=False`)
  - New grouped-mode `max_groups` validation with default threshold `20` and positive or `"unlimited"` override
  - Fix default (`"auto"`) `bin` calculation to Rice rule in all paths

Aligns template-side validation and forwarding for `Facet`, `Facet_Ncol`, `Max_Groups`, facet size hints, and overlay-only `multiple`.
- New template-layer APIs: `Facet`, `Facet_Ncol`, `Max_Groups`, `Element` (already exists in core-histogram function)
- `Figure_Width` / `Figure_Height` use `"auto"` defaults in facet mode while still use `8x6` in other paths
- More robust validation and error handling
  - Selective forwarding (`facet_ncol`, `facet_fig_width`, `facet_fig_height`, and `facet_tick_rotation` are forwarded only when `facet=True`, `max_groups` is forwarded only when `group_by` is active, `multiple` is forwarded only for grouped same-axis overlays)
  - Positivity check for more parameters (`fig_width / height`, `fig_dpi`, `facet_ncol`) 
  - String normalization for `multiple`, `element`, `stat`
- Fix layout issue with figure title with an adaptive algorithm

Adds focused tests

- Add dedicated tests for new helper `_derive_facet_geometry()`.
- Add dedicated tests for new facet path:
  - Smoke-path and output-structure coverage.
  - Figure-level label policy checks.
  - Numeric-annotation checks.
  - Shared-bin consistency tests for numeric and categorical cases.
  - Categorical facet `bins` handling.
  - Facet-size-hint validation and paired-hint checks.
  - Long-label geometry behavior.
- Add dedicated tests for new group-mode guardrails
  - New `max_groups` threshold checks (default, override, unlimited, and invalid) [[line 347-436](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R347)] 
  - External-`ax` guardrails. [line [505-551](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R505)]
- Other cleanup
  - Add `tearDown()` to histogram tests

## Testing

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
- `54 passed`

## Notes For Review

- Suggested review order:
  1. `src/spac/visualization.py` for the facet plotting path and grouped return-data contract.
  2. `src/spac/templates/histogram_template.py` for template-side validation, forwarding, and layout handling.
  3. `tests/test_visualization/test_histogram.py`, `tests/test_visualization/test_derive_facet_geometry.py`, and `tests/templates/test_histogram_template.py` for focused coverage.

*This pull request body is generated with the help of Codex using GPT-5.4 (xhigh)*

## Figures

More figures can be found here:

https://github.com/ramyap06/SCSAWorkflow-2025/blob/test/histogram-facet/notebooks/test_histogram_facet_light_template.ipynb

### Existing Grouped-Together Path

<img width="470.4" height="350.4" alt="image" src="https://github.com/user-attachments/assets/81b59b81-d73c-48bb-b9c6-24a00e105356" /> 

### Existing Grouped-Separate Path

<img width="470.4" height="350.2" alt="image" src="https://github.com/user-attachments/assets/fc24b074-e3e1-4318-90bb-57d4ae8c641f" />

### New Faceted path

<img width="470.2" height="384.4" alt="image" src="https://github.com/user-attachments/assets/730ab553-d1bb-493e-9f6a-b8b17a3e3927" />

--------

### Existing Grouped-Seperate Path with Long Labels

<img width="421.8" height="411.0" alt="image" src="https://github.com/user-attachments/assets/35073264-5bb0-416e-b510-f90c77087828" />

### New Faceted Path with Long Labels

<img width="715.6" height="645.2" alt="image" src="https://github.com/user-attachments/assets/c7ef5690-3a6b-4787-8c78-b16fb111a49a" />

--------

### New Faceted Path with More Groups

<img width="7163" height="6392" alt="image" src="https://github.com/user-attachments/assets/c75a9b23-ccb4-4f2c-a7f0-35c66b572624" />
