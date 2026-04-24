## Future Work

### Known Issue Fix

- How to deal with group-separate plotting (`group_by` is not `None`, `together=False`)
    - There are multiple layout issues in this paths, including
        - Incorrect output `hist_data` (currently repeatedly rewrote during the loop);
        - Overlapping label issues for long labels -> may reuse or imitate the facet geometry derivation logic.
        - Bad multiple plotting layout
    - Mousumi's opinion is that we may fully abandon this path?
- Naming issues with template parameters in JSON (not consistent with template tests as well as the blueprint, e.g. `"Table_"`). 

### Possible Enhancement (Need Evaluation)

- Confirm whether histogram template tests should remain I/O-oriented only or expand to handled-validation coverage.
- Blueprint follow-up: update blueprint with new facet controls and `stat="proportion"`, or align to a stricter blueprint/UI contract.
- UI follow-up for long axis labels: 
    - allow abbreviation of labels
    - allow label-level fontsize setting
- Output plot-related data in addition to the existing hist_data dataframe. e.g. add another column of the actual `stat` (e.g. `frequency`) in addition to the `count`.
- `kwargs` expansion:
    - Allow more seaborn `kwargs`;
    - Allow more values for existing `kwargs`;
    - A special case is `KDE`: this requires raw data plotting rather than pre-computed hist data by `calculate_histogram` function.
- External-`ax` support for facet mode.

### Possible Refactor (Need Evaluation)

- Refactor/simplify helper functions inside `histogram` function, and decide whether to relocate to module-level or `utils` folder (with unittests).
- Double-check facet geometry derivation flow in histogram function. Current derivation uses a complex algorithm.
- Double-check layout settings for facet mode in histogram template. Current algorithm uses magic numbers to solve overlapping between titles and subplots.