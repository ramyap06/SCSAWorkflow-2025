# PR Details With Diff Links

This file expands the current `## Details` section on
[PR #428](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428) with direct
links into the GitHub PR diff.

## 1. Facet Plotting and Histogram Return Contract

- Adds the new grouped `facet=True` plotting path in
  `spac.visualization.histogram()` and introduces
  `_derive_facet_geometry()` for adaptive facet layout.
  Links:
  [geometry helper, lines 407-536](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR407),
  [histogram signature and facet kwargs docs, lines 539-647](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR539),
  [facet validation and hint parsing, lines 756-856](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR756),
  [facet dispatch path, lines 1083-1158](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1083).

- Reuses one grouped histogram-bin table builder across grouped-together and
  facet paths so numeric and categorical grouped facets stay aligned.
  Links:
  [shared grouped histogram table builder, lines 904-968](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR904),
  [grouped-together shared-bin use, lines 1016-1049](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1016),
  [facet shared-bin use, lines 1101-1111](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1101).

- Standardizes the facet return `df` around grouped histogram-bin counts plus
  grouping metadata instead of raw plotting input.
  Links:
  [grouped histogram table return shape, lines 904-968](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR904),
  [facet branch assigns grouped histogram table to `hist_data`, lines 1103-1109](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1103),
  [final return contract, lines 1211-1214](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1211).

- Applies figure-level labels in facet mode and figure-level title handling
  instead of repeating axis labels and titles across panels.
  Links:
  [figure-level axis labels, lines 1193-1209](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1193),
  [facet title/layout handling in template, lines 352-406](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R352).

## 2. Guardrails, Bins, and Layout Hints

- Adds grouped-mode `max_groups` validation with default threshold `20` plus
  positive override and `"unlimited"` bypass behavior.
  Links:
  [core `max_groups` parsing, lines 802-820](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR802),
  [group-count validation, lines 1004-1014](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1004),
  [guardrail tests, lines 347-435](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R347).

- Rejects unsupported external `ax` usage for grouped-separate and facet
  layouts.
  Links:
  [core external-`ax` guardrail, lines 715-723](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR715),
  [external-`ax` tests, lines 491-551](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R491).

- Restricts `multiple` to grouped same-axis overlays and keeps grouped-separate
  / facet paths from carrying irrelevant overlay behavior.
  Links:
  [core `multiple` handling in grouped branches, lines 1016-1055](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR1016),
  [template selective forwarding, lines 279-295](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R279),
  [grouped-separate ignores `multiple` test, lines 617-631](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R617).

- Normalizes default-like `bins` values to the Rice-rule fallback for numeric
  histograms and keeps categorical grouped/facet bins aligned by category
  slots instead.
  Links:
  [core bins fallback, lines 730-754](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-20e13a5111550712bb4f52c47c970e21f21d82299e3a2c5622eecc915abfd4bfR730),
  [template bins handling, lines 158-187](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R158),
  [numeric bins fallback tests, lines 585-615](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R585),
  [categorical facet `bins` handling test, lines 1135-1171](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R1135).

## 3. Template Boundary and Layout Handling

- Exposes and validates `Facet`, `Facet_Ncol`, and `Max_Groups` at the
  template boundary.
  Links:
  [new template inputs, lines 96-119](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R96),
  [template `Max_Groups` and facet validation, lines 230-271](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R230).

- Uses `"auto"` figure-size defaults in facet mode, keeps explicit positive
  validation for figure size / DPI, and forwards facet hints only when the
  corresponding mode is active.
  Links:
  [figure size and DPI validation, lines 195-221](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R195),
  [assembled conditional forwarding, lines 279-309](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R279),
  [template smoke test now covers facet inputs, lines 50-65](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-def3b5c84148bb179383d10d24444ee08251d1ce6a5cd8ad307dbea6b56fec09R50).

- Keeps facet titles, rotated tick labels, and denser-grid spacing readable in
  the template wrapper.
  Links:
  [tick rotation anchor alignment, lines 352-357](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R352),
  [facet suptitle and panel titles, lines 359-386](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R359),
  [facet-aware tight layout, lines 391-406](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-6183d6d5d9566bc93571f393c82525bd2f2cbf057c53e285ca5fc991904e3d84R391).

## 4. Focused Tests Added or Expanded

- Adds dedicated tests for the new `_derive_facet_geometry()` helper.
  Links:
  [new helper test file, lines 1-95](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-c59aaf929c46fd41685f2813230c99c1c5d7561babca63bd52344a228cfbbe02R1).

- Adds dedicated tests for the new facet path:
  smoke-path coverage, output structure, figure-level label policy, and
  numeric-annotation support.
  Links:
  [facet smoke/path structure, lines 659-686](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R659),
  [figure-level labels/titles, lines 688-718](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R688),
  [numeric annotation facet support, lines 758-784](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R758).

- Adds dedicated tests for shared-bin consistency and categorical facet bin
  behavior.
  Links:
  [numeric shared-bin consistency, lines 974-1056](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R974),
  [categorical shared-bin consistency, lines 1058-1133](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R1058),
  [categorical facet `bins` handling, lines 1135-1171](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R1135).

- Adds dedicated tests for facet layout hints and long-label geometry.
  Links:
  [facet `facet_ncol` layout hints, lines 786-828](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R786),
  [facet size-hint validation, lines 830-874](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R830),
  [long-label geometry behavior, lines 895-935](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R895),
  [non-facet calls ignore facet-only hints, lines 937-972](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R937).

- Adds dedicated tests for new group-mode guardrails.
  Links:
  [new `max_groups` threshold checks, lines 347-435](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R347),
  [external-`ax` guardrails, lines 491-551](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R491).

- Other test cleanup.
  Links:
  [test `tearDown()` and small facet fixtures, lines 41-85](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428/files#diff-12828db6e930cc9b700eb2ed47fb195e3fca7ad99d89b39af0ac6779f2d41461R41).

## 5. Focused Verification Command

- `pytest tests/test_visualization/test_histogram.py tests/test_visualization/test_derive_facet_geometry.py tests/templates/test_histogram_template.py -q`
