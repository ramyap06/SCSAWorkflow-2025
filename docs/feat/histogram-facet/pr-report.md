# PR Report

Date: 2026-05-12

Reporter: Boqiang

This is a comprehensive PR report for the facet PR https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428.

## Contents

- Overview of facet PR
- Q&A For Facet PR
- Future Work

## Overview of Facet PR #428

### 1. Info

- PR Location: [#428](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/428) 
- Previous PR: [#328](https://github.com/FNLCR-DMAP/SCSAWorkflow/pull/328)
- More implementation details: [pr-details.md](https://github.com/ramyap06/SCSAWorkflow-2025/blob/ed5e80ca076bed61b05b20193bdb30212373b777/docs/feat/histogram-facet/pr-details.md)
- Testing notebook: [test_histogram_facet_light_template.ipynb](https://github.com/ramyap06/SCSAWorkflow-2025/blob/ed5e80ca076bed61b05b20193bdb30212373b777/docs/feat/histogram-facet/test_histogram_facet_light_template.ipynb)

### 2. Main Changes

#### (a) New Facet Path

facet-related APIs, helper functions, adaptive geometry/layout

- New APIs: 
  - flag in `histogram()` : `facet`
  - kwargs in `histogram()` : `facet_ncol`, `facet_figure_width`, `facet_figure_height`, `facet_tick_rotation`
  - in template: `Facet`, `Facet_Ncol`(`"auto"`)

- Update API contracts:
  - `Figure_Width`, `Figure_Height` allow `"auto"` when `Facet=True`

- Adaptive facet geometry/layout:
  - New module-level helper `spac.visualization._derive_facet_geometry()` for adaptive facet layout
  - `tight_layout` based on facet geometry

- Several inner helpers in `histogram()` (`_parse_optional_number()`, `build_grouped_histogram_table()`, `compute_max_tick_label_length()`, `resolve_hist_axis_labels()`)

#### (b) Guardrails

group number threshold (`Max_Groups`), seaborn-kwarg exposure (`Elements`), rejection (external-`ax`, non-together-`multiple`), validation

- New APIs:
  - kwarg in `histogram()`: `max_groups` (`20`/`"unlimited"`), `shrink`, `alpha`
  - in template: `Max_Groups`(`20`, when `Group_by` is set), `Elements`(`"bars"`)

- Update API contracts:
  - `bins`: seaborn-scope `"auto"`/`"None"`/`""` now fallback to default Rice-rule calculation instead of seaborn `"auto"`
  - `ax`: rejects external `ax` now in grouped-separated and facet paths
  - `multiple`: automatic dropping this kwarg in non-together paths now

- Validation/Normalization:
  - Validates postivitiy for `fig_width`, `fig_height` and `figure_dpi`
  - Light normalization of seaborn-kwargs `Element`, `Stat` and `Multiple`
  - Selective forwarding of `Max_Groups`, four `Facet_*`, `Multiple`

#### (c) Miscellaneous Work

- Refactor to avoid opening figures not used
- Expands and revises some docstrings/comments
- Numerous unittests for facet:
  - smoke test, template smoke test
  - figure-level labels and titles
  - numeric annotation support
  - new APIs of layout hints and size-hint
  - long-label layout issue check
  - shared-bin consistency
  - `_derive_facet_geometry()` tests
- Numerous unittests for other new features:
  - max_groups threshold check
  - external-ax rejection check
  - new API contracts for `bins`, `multiple` etc


## Q&A For Facet PR

Q1. There is a group-separate plotting, with inconsistent scales and several layout issues. Mousumi just dropped it on the Shiny front-end. Do we still keep it and then fix it in the package?

- We remove it. Do it in a new PR.

Q2. Naming issues - inconsistent naming across templates, blueprint, shiny, template tests. Which is the fixed standard?

- Email George with examples. We fix naming in templates and refer to it. No need to worry about blueprint (for me)

Q3. Histogram template unittests: only I/O?

- Can test more, but only template-specific. Write new tests only when bugs are discovered to ensure meaningful coverage.

Q4. Do I update blueprint?

- No. No need to handle that.

Q5. Abbreviation of labels, label-level fontsize setting (current is overall) - do we incorporate it into the package, or leave them as front-end adjustment?

- Yes to package. No for me - can leave them for future developers (e.g. high school students!)

Q6. Current output data is just hist_data. Do we consider more data like the actual stat form (frequency, proportion, etc), and more, to the data?

- We can! Leave this to future developer.

Q7. Will people need external `ax` support for? I see in template it is fully abandoned

- Still keep it in the core function. Do not reject them all. For facet we cannot accept it but no need to break the process. (double-check, and if needed revise it in the new PR)

Q8. Do you think my current way of writing helpers good? I feel like heavy?

- It is ok?

Q9. I will need this facet helper for datashader density. What should I do? Do I put in `utils` folder?

- Yes. Put it in utils. Do it in a new PR.

Q10. I used a complicated algorithm to compute layout (ratio of height and width based on rotation & label length, facet_ncol, etc.). Good or bad?

- N/A. We will see.


## Future Work

### Immediate Next Steps

- Follow-up for the naming inconsistency between template and tests (email George)
- Remove the deprecated plotting mode for together=False, facet=False
- Allow auto-filter to most frequent groups with notification (rather than rejecting)
- Move `derive_facet_geometry` to utils (and possible improvement)
- Do not reject `ax` directly. Provide it in core function (for facet we can just provide our plotting)
- Add unittests for additional functionality on template

### More Ideas

- Allow abbreviation of labels, label-level fontsize setting (current examples are in Shiny side, `feat_vs_anno` tab (hierachical heatmap)) for long-label issues
- More plot-related data in output dataframe, e.g. frequency/proportion if specified in `stat`
- `kwargs` expansion:
  - Allow more seaborn `kwargs`;
  - Allow more values for existing `kwargs`;
  - A special case is `KDE`: this requires raw data plotting rather than pre-computed hist data by `calculate_histogram` function.
- Possible simplification/reloation for helper functions (need evaluation)
- Possible simplification for facet geometry/layout derivation (need evaluation). Current way is driven by AI
