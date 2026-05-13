## Description

This PR completes the histogram facet work in `SCSAWorkflow` by turning the earlier facet prototype into a more consistent plotting path across `spac.visualization.histogram()` and `histogram_template.py`.

**This PR adds faceted histogram support with adaptive layout, shared bins, grouped-mode guardrails, and a consistent returned histogram dataframe contract.**

## Related Issue/PR

This PR depends on PR #328.

## Key Changes

### 1. Facet Plotting and Histogram Return Contract

- Added `facet=True` support in `spac.visualization.histogram()` for grouped histograms.
- Enforced the facet guardrails that require `group_by` and disallow `together=True`.
- Added `_derive_facet_geometry()` to derive facet layout from group count, size hints, and rotated long-label burden.
- Reused a shared grouped histogram-bin table for grouped-overlay and facet paths so numeric and categorical facets stay aligned.
- Standardized the facet return `df` around grouped histogram-bin counts instead of raw plotting input.
- Added figure-level x/y labels for facet mode and blocked unsupported external-`ax` usage for grouped-separate and facet layouts.

### 2. Binning, Layout Hints, and Grouped Guardrails

- Normalized default-like `bins` inputs (`None`, `"auto"`, `"none"`, `""`) to the Rice-rule fallback for numeric histograms.
- Kept categorical grouped/facet paths aligned by preserving shared category slots and ignoring numeric-style `bins` changes there.
- Added grouped-mode `max_groups` validation with default threshold `20` and `"unlimited"` override support.
- Parsed `facet_ncol`, `facet_fig_width`, `facet_fig_height`, and `facet_tick_rotation` only when `facet=True`.
- Ignored grouped-only and facet-only hints when their corresponding modes are inactive instead of validating irrelevant inputs.
- Restricted `multiple` to grouped same-axis overlays so grouped-separate and facet paths do not carry irrelevant overlay behavior.

### 3. Template Boundary and Layout Handling

- Exposed and validated `Facet`, `Facet_Ncol`, and `Max_Groups` in `run_from_json()`.
- Treated `Figure_Width` / `Figure_Height` as `"auto"` defaults in facet mode so core facet geometry can determine final sizing when explicit hints are not provided.
- Tightened figure-size validation so explicit zero values fail fast instead of bypassing checks.
- Kept seaborn-native `element` and `stat` controls lightly normalized without adding extra SPAC-only restrictions.
- Adjusted facet title and layout handling so faceted figure titles and rotated tick labels remain readable across denser grids.

### 4. Tests and Documentation

- Added a dedicated test file for `_derive_facet_geometry()`.
- Expanded histogram tests for:
  - facet smoke behavior and output structure
  - figure-level label policy
  - numeric and categorical shared-bin consistency
  - numeric-annotation facet support
  - facet-size hints and paired-hint validation
  - long-label geometry adjustment
  - `max_groups` guardrails
  - external-`ax` guardrails
  - categorical facet `bins` behavior
- Updated histogram/template docstrings and local PR tracking docs to match the finalized facet contract.

## Files Modified

- `src/spac/visualization.py`
- `src/spac/templates/histogram_template.py`
- `tests/test_visualization/test_histogram.py`
- `tests/test_visualization/test_derive_facet_geometry.py`
- `tests/templates/test_histogram_template.py`
- `src/spac/utils.py` for minor cleanup

## Verification Results

- [x] Focused histogram/helper/template test suite passing
  - `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
  - `54 passed, 1 warning`
- [x] Facet mode returns multi-axis output with figure-level labels
- [x] Shared bin alignment verified for numeric and categorical grouped facets
- [x] Non-facet and non-grouped paths ignore irrelevant grouped/facet hints
- [x] `max_groups`, facet-size-hint pairing, and external-`ax` guardrails are covered by focused unit tests

**Ready for Review**
