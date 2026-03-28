# Histogram Facet Refactoring Plan (Updated)

## Problem Statement
The histogram facet feature in `src/spac/visualization.py` has been implemented but needs refactoring before code review. Multiple concerns need to be addressed:

1. **Duplicated global_bin_edges computation** - Same logic exists in both `together` mode and `facet` mode
2. **Parameter distribution** - 4 new facet parameters may not be appropriately distributed between bare function and template
3. **Bug at line 723** - `ax_array.flatten()` replaces title-setting code in annotation histogram branch
4. **Test inconsistency** - Facet tests expect per-axis labels but code uses `supxlabel/supylabel`
5. **Axes handling complexity** - Multiple axes variables may be redundant

---

## Analysis Summary

### Current Code Structure (from diff)
- **Lines 657-662**: Global bin edge computation in `together` mode
- **Lines 769-774**: Identical computation duplicated in `facet` mode  
- **Line 723**: Bug - `ax_array = ax_array.flatten()` should be `ax_i.set_title(f'{groups[i]}')`
- **New parameters via kwargs**: `facet_ncol`, `facet_vertical_threshold`, `facet_height`, `facet_aspect`
- **Axes variables**: `ax`, `axs`, `ax_array`, `axes` (line 849: `axes = axs if isinstance(axs, (list, np.ndarray)) else [axs]`)

### Codebase Pattern Findings
1. **Helper function organization**:
   - **Inner functions**: Computation-specific logic used only within one function (e.g., `calculate_histogram()`)
   - **Module-level private functions**: Logic shared across multiple functions or anticipated to be reusable (e.g., `_prepare_spatial_distance_data()`)
   - **utils.py**: General-purpose utilities (validation, regex, data manipulation) - NOT visualization-specific

2. **Template vs bare function separation**:
   - Bare functions: core computation, statistical correctness, plotting semantics
   - Templates: figure sizing, DPI, legend positioning, axis rotation, titles, parameter parsing from JSON
   
3. **Other templates already handle**: `fig.set_size_inches()`, `fig.set_dpi()`, `sns.move_legend()`, `ax.tick_params(rotation=...)`

---

## Refactoring Recommendations

### 1. Extract Global Bin Edges Computation (Code Deduplication) ⭐ HIGH PRIORITY

**Recommendation**: Create a **module-level private helper** `_compute_global_bin_edges()` in `visualization.py` since faceting is anticipated to be implemented in other plot types (boxplot, scatter).

**Location**: Place around line 400 in `visualization.py`, before the `histogram()` function definition. This follows the pattern of other private helpers like `_prepare_spatial_distance_data()`.

**Proposed signature**:
```python
def _compute_global_bin_edges(data_series, bins):
    """
    Compute consistent bin edges across all data for aligned histograms/facets.
    
    Parameters
    ----------
    data_series : pd.Series
        The data to compute bin edges for.
    bins : int or sequence
        Number of bins (for numeric data) or bin specification.
        
    Returns
    -------
    array-like
        Bin edges for numeric data, or unique categories for categorical data.
    """
    if pd.api.types.is_numeric_dtype(data_series):
        return np.histogram_bin_edges(data_series, bins=bins)
    else:
        return data_series.unique()
```

**Rationale**:
- **Reusability**: Faceting will likely be added to boxplot, scatter, and other plots → module-level enables reuse
- **Testability**: Can be tested independently if needed
- **Pattern consistency**: Follows existing module-level helpers in visualization.py (`_prepare_spatial_distance_data()`, etc.)
- **Not utils.py**: This is visualization-specific logic, not a general utility

**Trade-off considered**: Inner function would follow the pattern of `calculate_histogram()`, but since you anticipate faceting elsewhere, module-level is more appropriate.

**Action items**:
- [ ] Create `_compute_global_bin_edges()` as module-level function near line 400
- [ ] Replace duplicated code at lines 657-662 (together mode) with function call
- [ ] Replace duplicated code at lines 769-774 (facet mode) with function call
- [ ] Add docstring following numpy style guide

---

### 2. Parameter Distribution Between Bare Function and Template

**Analysis of current facet parameters**:

| Parameter | Current Location | Recommended Location | Rationale |
|-----------|-----------------|---------------------|-----------|
| `facet` (bool) | `histogram()` | **Keep in `histogram()`** | Core behavior switch, like `together` |
| `facet_ncol` | kwargs in `histogram()` | **Move to template** | Layout config, similar to `fig_width`/`fig_height` |
| `facet_vertical_threshold` | kwargs in `histogram()` | **Move to template** | Layout heuristic, UI concern |
| `facet_height` | kwargs in `histogram()` | **Move to template** | Figure sizing, already handled by template |
| `facet_aspect` | kwargs in `histogram()` | **Move to template** | Figure sizing, already handled by template |

**Reasoning**:
- (a) **Pattern from other templates**: Bare functions handle core logic; templates handle presentation/layout
- (b) The 4 layout parameters (`facet_ncol`, `facet_vertical_threshold`, `facet_height`, `facet_aspect`) are analogous to `fig_width`, `fig_height`, `fig_dpi` which are already in templates
- (c) Only the `facet=True/False` toggle belongs in the bare function (like `together`)

**Proposed approach**:
- **Option A (Simpler)**: Keep parameters in `histogram()` for now, add them to `histogram_template.py` as passthrough. This maintains backward compatibility.
- **Option B (Cleaner)**: Move FacetGrid creation to template layer, have bare function return data needed for faceting. More complex refactor.

**Recommended**: Option A for this iteration. The parameters can stay in kwargs but the template should explicitly handle them and pass them through.

**Action items**:
- [ ] Add `facet_ncol`, `facet_vertical_threshold`, `facet_height`, `facet_aspect` parameters to `run_from_json()` in histogram_template.py
- [ ] Pass these through to the `histogram()` call
- [ ] Document the design decision: facet parameters are in kwargs but should be controlled by template layer

---

### 2c. UI Functionality (Axis Abbreviation/Rotation) Location

**Question**: Should axis abbreviation and rotation move from Shiny frontend to `run_from_json` backend?

**Answer**: **Yes, move to template layer** (`histogram_template.py`)

**Rationale**:
1. **Mentor's guidance aligns with pattern**: Templates are the "platform-agnostic" layer between bare functions and platform-specific UIs (Shiny, Galaxy, CLI)
2. **Current precedent**: `histogram_template.py` already has `x_rotate` parameter (line 114) which applies rotation at line 251
3. **DRY principle**: If Shiny, Galaxy, and CLI all need axis rotation, implement once in template
4. **Galaxy portability**: Galaxy workflows call templates, not Shiny code

**Action items**:
- [ ] Add axis abbreviation logic to `histogram_template.py` (after calling `histogram()`)
- [ ] Ensure rotation logic is centralized in template (already present at line 251)
- [ ] Other platforms (Galaxy, CLI) automatically gain these features

---

### 3. Simplify Axes Handling

**Investigation**: Can `fig.axes` replace `ax_array`?

**Finding**: **Partially yes, but with caveats**

**Current code flow**:
```python
# Line 700-710: Non-facet grouped mode
fig, ax_array = plt.subplots(n_groups, 1, figsize=(5, 5 * n_groups))
if n_groups == 1:
    ax_array = [ax_array]
else:
    ax_array = ax_array.flatten()

# Line 777-793: Facet mode uses FacetGrid
hist = sns.FacetGrid(...)
# Line 821: Extract axes
axs.extend(hist.axes.flat)
```

**Analysis**:
1. **`fig.axes`** returns list of all axes on a figure, so theoretically `fig.axes` could replace `ax_array.flatten()` 
2. **However**, `plt.subplots()` returns axes in predictable order; `fig.axes` may include unexpected axes (e.g., colorbars)
3. **FacetGrid** already provides `hist.axes.flat` which is idiomatic

**Recommended simplifications**:
1. **For non-facet grouped mode** (lines 700-748): `fig.axes` is safe to use since we create a simple subplot grid
2. **For facet mode**: Keep using `hist.axes.flat` (already clean)
3. **Consolidate axes normalization**: Remove `axes = axs if isinstance(...)` pattern at line 849; just ensure `axs` is always a list

**Proposed cleaner pattern**:
```python
# Non-facet grouped mode
fig, ax_array = plt.subplots(n_groups, 1, figsize=(5, 5 * n_groups))
# Use fig.axes directly instead of manual flatten
for i, ax_i in enumerate(fig.axes):
    ...
    axs.append(ax_i)
```

**Caution**: Must verify no other elements (legends, annotations) add extra axes to fig.axes. The current explicit approach is defensive. This simplification is **lower priority** than items 1-2.

**Action items**:
- [ ] Verify `fig.axes` doesn't include unexpected axes in `plt.subplots()` case
- [ ] If safe, replace `ax_array.flatten()` with `fig.axes`
- [ ] Consider keeping defensive approach if uncertainty exists

---

### 4. Fix Bug at Line 723 ⭐ HIGH PRIORITY

**Problem**: Inside the non-facet grouped plotting loop, the `else` branch (for annotation histograms) has:
```python
else:
    ax_array = ax_array.flatten()
```

This is nonsensical—it re-flattens an already-flattened array inside a loop and doesn't set a title.

**Original code** (from FNL-dev branch):
```python
else:
    ax_i.set_title(f'{groups[i]}')
```

**Explanation**: 
- Feature histograms get titles like `"Group A with Layer: Original"`
- Annotation histograms should get simpler titles like `"Group A"` (no layer info since annotations don't have layers)

**Action items**:
- [ ] Replace line 723 with: `ax_i.set_title(f'{groups[i]}')`
- [ ] Verify titles appear correctly for annotation-based grouped histograms

---

### 5. Test Inconsistency (DEFERRED)

**Issue**: The facet test (`test_facet_plot`) expects per-axis labels:
```python
self.assertIn('marker1', axis.get_xlabel(), ...)
```

But the code clears per-axis labels and uses `fig.supxlabel()` and `fig.supylabel()` instead (lines 855-862, 882-883).

**Decision**: **Defer until after implementation is complete**. Once the core functionality is finalized, you'll:
1. Decide whether per-axis labels or super-labels are the desired behavior
2. Update tests to match the implementation
3. Add new unit tests for your contributions (global bin consistency, facet layout options, etc.)

**Action items** (deferred):
- [ ] Review label strategy and decide on final approach
- [ ] Update `test_facet_plot` expectations to match implementation
- [ ] Add tests for: global bin edge consistency, facet layout parameters, user-supplied ax behavior

---

## Priority Order

1. **HIGH PRIORITY**: 
   - Fix bug at line 723 (broken title setting)
   - Extract `_compute_global_bin_edges()` module-level function
   
2. **MEDIUM PRIORITY**: 
   - Add facet parameters to template and document design rationale
   - Plan for axis abbreviation/rotation in template layer
   
3. **LOW PRIORITY**: 
   - Investigate `fig.axes` simplification (test carefully before changing)
   
4. **DEFERRED**: 
   - Update facet tests after implementation finalized

---

## Implementation Checklist

**Phase 1: Bug Fixes and Core Refactoring**
- [ ] Fix line 723 bug: restore `ax_i.set_title(f'{groups[i]}')`
- [ ] Create `_compute_global_bin_edges()` at module level (line ~400)
- [ ] Replace together mode bin computation (lines 657-662) with helper call
- [ ] Replace facet mode bin computation (lines 769-774) with helper call
- [ ] Run tests: `python -m pytest tests/test_visualization/test_histogram.py`

**Phase 2: Template Layer Enhancement**
- [ ] Add `facet_ncol`, `facet_vertical_threshold`, `facet_height`, `facet_aspect` to `histogram_template.py`
- [ ] Pass these parameters through to `histogram()` call
- [ ] Document design decision in template docstring
- [ ] Plan axis abbreviation feature for template (if not already present)

**Phase 3: Testing and Cleanup (Deferred)**
- [ ] Update `test_facet_plot` to match final label strategy
- [ ] Add test for global bin edge consistency across facets
- [ ] Add test for facet layout parameters
- [ ] Consider: test for `fig.axes` simplification if implemented

---

## Notes for Implementation

- Run tests after each change: `python -m pytest tests/test_visualization/test_histogram.py`
- The existing test `test_facet_plot` may fail until Phase 3 (expected)
- Focus on Phase 1 (bug fixes + refactoring) before moving to template enhancements
