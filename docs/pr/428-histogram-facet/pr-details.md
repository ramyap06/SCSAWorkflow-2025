# PR Details

This file is the reviewer appendix for [pr-summary.md](./pr-summary.md). 

## 1. Core Facet Introduction

### Files related

- `src/spac/visualization.py`

### Notable changes

- Adds the grouped `facet=True` plotting path in `spac.visualization.histogram()`
- Introduces new APIs to `histogram()`
  - facet mode flag: `facet`
  - facet geometry hints (kwargs): `facet_ncol`, `facet_figure_width`, `facet_figure_height`, `facet_tick_rotation`
- Introduces new module-level helper function `spac.visualization._derive_facet_geometry()` for adaptive facet layout
- Introduces several reusable inner helpers in `histogram()`, mostly extracted from existing paths in order to be applied to facet path:
  - `_parse_optional_number()`: validate and parse numbers with specified strings (e.g. `"auto"`, `"None"`, `"unlimite"`)
  - `build_grouped_histogram_table()`: compute shared histogram table `df` and `bin_edges` for grouped data
  - `compute_max_tick_label_length()`: compute maximum length of tick labels as a hint for adaptive facet geometry derivation
  - `resolve_hist_axis_labels()`: determine axis labels based on scale/stat settings

### Details

- Adds a grouped `facet=True` plotting path in `histogram()`.
  - Introduces a new facet mode flag `facet` (bool default `False`, [line R541, R586-590](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR541)) with validation ([line R756-763](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR756))
  - Introduces new facet-hint kwargs ([line R638-648](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR638)), with validation and parsing ([line R804-808](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR804), [R822-857](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR822)) using an inner helper `_parse_optional_number()` ([line 764-801](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR764))
    - `facet_ncol` (postivite integer, allow `"auto"`/`"None"`/`""` which fallback to default `None`)
    - `facet_fig_width`, `facet_fig_height` (positive integer, default to `None`, with pair semantic validation)
    - `facet_tick_rotation` (float, default to `0.0`, parsed by `% 360.0`)
  - Facet plotting path is at [line R1083-1158](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1083), structure:
    - Derive facet geometry based on group count and layout hints (line R1086-1100)
    - Compute histogram data and shared bins (line R1101-1112)
    - Use `seaborn.FacetGrid()` to plot faceted histogram (line R1113-1131)
    - Set labels and titles (line R1132-1144)
    - Post-plotting processing: set layout margins and figure size, prepare axs data (line R1145-1158)

- Allows adaptive facet geometry derivation
  - Introduces a new module-level geometry helper function `_derive_facet_geometry()` based on above `"facet_*"` hints and other default settings for adaptive facet layout ([line R406-538](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR407))
  - Introduces an inner helper function  `compute_max_tick_label_length()` to compute maximum tick label length as an additional hint for facet layout geometry derivation (line [R970-981](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR970))

- Reuses one grouped histogram-bin table builder across grouped-together and facet paths
  - Extracts an inner helper `build_grouped_histogram_table()` ([line R904-969](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR904)) from grouped-together (`together=True`) path (from [line L639-655](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfL639)) to compute concated histogram-bin table and shared_bin_edges
  - Expands and reuses the helper for both facet path ([line R1101-1112](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1101)) and group-together path ([line R1017-1029](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1017))

- Standardizes the facet return `df` around grouped histogram-bin counts plus grouping metadata using the above inner helper (previous PR return the raw data in facet path)

- Streamlines label settings
  - Extracts reusable axis label determining inner helper `resolve_hist_axis_label()` ([line R982-997](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR982)) from [line L693-715](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfL693), [L738-760](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfL738) and applies to axis label setting at [line R1184-1209](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1184)
  - Set figure-level axis labels in facet mode instead of axes-level ([line R1199-1209](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1193))

## 2. Guardrails and Binning

Notable changes:

- Introduce new kwarg: `max_groups`
- Document known seaborn kwargs that are already exposed in template APIs: `shrink`, `alpha`
- Update API contracts:
  - `bins`: seaborn-scope `"auto"` now fallback to default Rice-rule calculation, as well as default-like string `"None"`, `""`, to avoid duplicate binning
  - `ax`: rejects external `ax` now in grouped-separated and facet paths
  - `multiple`: automatic dropping this kwarg in non-together paths now

Details:

- Adds a new grouped-mode kwarg `max_groups` to avoid performance issues or unreadable plots
  - Introduces a new kwarg `max_groups` ([line R631-637](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR631)) in `histogram()`
  - Parses `max_groups` with default threshold `20`, positive overrides, and `"unlimited"` bypass behavior ([line R802-803, R809-821](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR802))
  - Validate group count `n_groups` with `max_groups` ([line R1007-1014](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1004))

- Rejects unsupported external `ax` usage for grouped-separate and facet layouts ([line R715-722](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR715)), with docstrings update ([line R576-577](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR576))

- Restricts `multiple` to grouped same-axis overlays only ([line R1052-1053](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1052)) with docstrings update ([line R567-568, 595-596](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR567))

- Restricts `bins` contracts
  - Normalizes default-like `bins` values (`"auto"`/`"None"`/`""`) to the Rice-rule fallback for numeric histograms ([line R746-753](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR746), [R1031](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1031))
  - Keeps categorical grouped/facet bins aligned by category slots instead ([line R730](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR730), [R1028](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1028))
  - Docstring update ([line R627-628](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR627))

- Document seaborn kwargs `shrink`, `alpha` in `histogram()` docstrings ([line R608-611](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR608))

## 3. Miscellaneous Work

### Files related

- `visualization.py`
- `utils.py`

### Details

In `visualization.py`:

- Postpones figure creation logic to plotting branches to avoid unused figures ([line L552-553](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfL552), [R1018-1019](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1018), [R1161-1164](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1161)).
- Expands and normalizes several docstrings and comments.

In `utils.py`:

- Whitespace cleaning (side effect of a Codex-driven cleaning for all related files)

## 4. Template Boundary

### Files related

- `histogram_template.py`

### Notable changes

- New template APIs: `Facet`, `Facet_Ncol`, `Max_Groups`, `Elements`
- API contracts update: `Figure_Width`, `Figure_Height` (allow `"auto"` when `Facet=True`)

### Details

- New template-level APIs for histogram.
  - New `Facet`, `Facet_Ncol` for facet mode with defaults `Facet=False`, `Facet_Ncol="auto"` ([line R118-119](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R118)), and validation (line [R248-272](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R248))
  - New guardrail `Max_Groups` - threshold for number of groups - for all group-related paths (`Group_by` is set) with defaults `Max_Groups=20` ([line R99-100, R302](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R99)), and validation ([R230-247](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R230))

- Enhances `Figure_Width`, `Figure_Height` and `Figure_DPI` contracts and validation.
  - Sets `"auto"` for `fig_width`, `fig_height` defaults in facet mode (to allow adaptive facet geometry derivation) ([line R102-103](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R102), [R195-211](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R195))
  - Validates postivitiy for `fig_width`, `fig_height` and `figure_dpi` with logging ([line R212-222](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R212))
  - Adds guardrails to Figure Size & DPI setting ([R316-318, R320](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R316))

- Enhances seaborn kwargs handling.
  - Exposes seaborn-kwarg `Element` to template (with default `"bars"`) ([line R111](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R111)) that is already documented in core `histogram()` docstrings
  - Light normalization of seaborn-kwargs `Element`, `Stat` and `Multiple` ([line R189-194](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R189))
  - Expands `Bins` contract documentation ([line R158-160](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R158))

- Selective forwarding and kwargs assembly ([line R279-296, R307-308](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R279))
  - Forwards `Max_Groups` only in grouped plot paths (`Group_by` is set)
  - Forwards facet hints (four `Facet_*`) only when facet mode is active
  - Forwards seaborn-kwarg `Multiple` only when group-together mode is active

- Other layout issues
  - Fixes overlapping issue caused by long x-ticklabel rotation ([line R354-357](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R354))
  - Fixes overlapping issue caused by default tight_layout ([line R391-406](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R391))
  - Sets facet mode titles ([line R376-382, R385](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/changes#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R376))


## 5. Focused Tests

Tests include but not limited to (this list is generated by GPT-5.4, I haven't checked very carefully. @ me if you have any questions)

- Expands histogram tests (`tests/templates/test_histogram_template.py`) for the new facet path for:
  - [facet smoke/path structure](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R659),
  - [figure-level labels and titles](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R688),
  - [numeric annotation facet support](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R758),
  - [facet layout hints and size-hint validation](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R786),
  - [long-label geometry behavior](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R895),
  - [shared-bin consistency and categorical facet bins](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R974).

- Adds dedicated tests for the new grouped guardrails.
  - [new `max_groups` threshold checks](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R347),
  - [external-`ax` guardrails](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R491),

- Adds dedicated tests for update seaborn-kwargs behavior
  - [numeric bins fallback tests](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R585),
  - [categorical facet `bins` handling test](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R1135).
  - [grouped-separate ignores `multiple` test](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R617).

- Adds test cleanup and small facet fixtures, see [here](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R41),

- Adds dedicated tests for `_derive_facet_geometry()`, see [tests/test_visualization/test_derive_facet_geometry.py](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-c59aaf929c46fd41685f2813230c99c1c5d7561babca63bd52344a228cfbbe02R1).

- Adds template smoke test with facet inputs, see [tests/templates/test_histogram_template.py](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-def3b5c84148bb179383d10d24444ee08251d1ce6a5cd8ad307dbea6b56fec09R50).

## 6. Suggested Review Order

1. `src/spac/visualization.py`
2. `src/spac/templates/histogram_template.py`
3. `tests/test_visualization/test_histogram.py`
4. `tests/test_visualization/test_derive_facet_geometry.py`
5. `tests/templates/test_histogram_template.py`

## 7. Focused Verification

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
- `54 passed`

## 8. Figures

- Existing notebook reference:
  [test_histogram_facet_light_template.ipynb](./test_histogram_facet_light_template.ipynb)
- Current GitHub PR body on [PR #428](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428) already includes representative grouped vs. faceted comparison figures.
