## Related PR

This PR depends on PR #328.

## Summary

This PR finalizes histogram facet support in `SCSAWorkflow` by adding a faceted grouped-histogram path in `spac.visualization.histogram()` and aligning `histogram_template.py` with the same facet/grouped contract.

## Changes

- Adds `facet=True` grouped histogram support with shared bins and shared category slots, adaptive facet geometry, figure-level labels, grouped count-table returns, and grouped-mode guardrails.
- Aligns template-side validation and forwarding for `Facet`, `Facet_Ncol`, `Max_Groups`, facet size hints, and overlay-only `multiple`, while keeping grouped-only and facet-only hints inactive outside their modes.
- Adds focused tests for `_derive_facet_geometry()`, facet behavior, bins fallback, shared-bin consistency, `max_groups`, and external-`ax` guardrails.

## Testing

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
- `54 passed`

## Notes For Review

- Suggested review order:
  1. `src/spac/visualization.py` for the facet plotting path, shared-bin behavior, and grouped return-data contract.
  2. `src/spac/templates/histogram_template.py` for template-side validation, forwarding, and layout handling.
  3. `tests/test_visualization/test_histogram.py`, `tests/test_visualization/test_derive_facet_geometry.py`, and `tests/templates/test_histogram_template.py` for focused coverage.
- More implementation details can be found here: [pr-summary-details.md](https://github.com/ramyap06/SCSAWorkflow-2025/blob/073edd96e42531c9f8372fdd69167fdd194d049a/docs/pr/428-histogram-facet/pr-details.md).
- Here is a jupyter notebook with plenty of tests: [test_histogram_facet_light_template.ipynb](https://github.com/ramyap06/SCSAWorkflow-2025/blob/073edd96e42531c9f8372fdd69167fdd194d049a/docs/pr/428-histogram-facet/test_histogram_facet_light_template.ipynb). Some output figures are attached below.
- Here is a list of possible future work: [future-work.md](https://github.com/ramyap06/SCSAWorkflow-2025/blob/073edd96e42531c9f8372fdd69167fdd194d049a/docs/pr/428-histogram-facet/future-work.md)

*This pull request body is generated with the help of Codex using GPT-5.4 (xhigh)*

## Selected Figures

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
