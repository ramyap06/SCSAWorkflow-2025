# Histogram Facet Refactoring Plan (Updated)

## Problem Statement
The histogram facet feature in `src/spac/visualization.py` has been implemented but needs refactoring before code review. Multiple concerns need to be addressed:

1. **Duplicated global_bin_edges computation** - Same logic exists in both `together` mode and `facet` mode
2. **Parameter boundary** - Facet layout controls should be simplified and split cleanly between template and bare function
3. **Bug at line 723** - `ax_array.flatten()` replaces title-setting code in annotation histogram branch
4. **Test inconsistency** - Facet tests expect per-axis labels but code uses `supxlabel/supylabel`
5. **Axes handling complexity** - Multiple axes variables may be redundant

---

## Analysis Summary

### Current Code Structure (from diff)
- **Lines 657-662**: Global bin edge computation in `together` mode
- **Lines 769-774**: Identical computation duplicated in `facet` mode  
- **Line 723**: Bug - `ax_array = ax_array.flatten()` should be `ax_i.set_title(f'{groups[i]}')`
- **Current facet kwargs in `histogram()`**: `facet_ncol`, `facet_vertical_threshold`, `facet_height`, `facet_aspect`
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
- [x] Create `_compute_global_bin_edges()` as module-level function near line 400
- [x] Replace duplicated code at lines 657-662 (together mode) with function call
- [x] Replace duplicated code at lines 769-774 (facet mode) with function call
- [x] Add docstring following numpy style guide

---

### 2. Parameter Boundary Between Bare Function and Template

**Analysis of current facet parameters**:

| Parameter | Current Location | Recommended Location | Rationale |
|-----------|-----------------|---------------------|-----------|
| `facet` (bool) | `histogram()` | **Expose via template** | Core behavior switch, like `together` |
| `facet_ncol` | kwargs in `histogram()` | **Expose via template** | Intuitive layout control users understand |
| `Figure_Width`, `Figure_Height`, `Figure_DPI` | template | **Template-only user contract** | Existing and familiar figure sizing contract |
| `target_fig_width`, `target_fig_height` | internal kwargs to `histogram()` | **Internal-only hints** | Needed to derive facet panel geometry from final figure size |
| `facet_vertical_threshold`, `facet_height`, `facet_aspect` | kwargs in `histogram()` | **Remove from public histogram API** | Overlapping/technical knobs that complicate UX |

**Reasoning**:
- (a) **Pattern from other templates**: Bare functions handle core logic; templates handle presentation/layout
- (b) User-facing API should remain simple for biology users (`facet` + `facet_ncol` + figure-level size)
- (c) `facet_height`/`facet_aspect` and threshold behavior are technical tuning knobs that cause confusion and conflict with figure-level sizing

**Proposed approach (selected)**:
- Keep FacetGrid creation in `histogram()`.
- Expose only `facet` and `facet_ncol` at template level.
- Pass template figure size to `histogram()` as internal hints (`target_fig_width`, `target_fig_height`).
- Derive panel geometry internally and do not expose `facet_height`/`facet_aspect`/`facet_vertical_threshold`.

**Action items**:
- [ ] Add `facet` and `facet_ncol` parameters to `run_from_json()` in histogram_template.py
- [ ] Pass only user-facing facet controls through template into `histogram()`
- [ ] Pass `target_fig_width` and `target_fig_height` from template into `histogram()` as internal kwargs
- [ ] Remove `facet_vertical_threshold`, `facet_height`, and `facet_aspect` from histogram docstring/API contract
- [ ] Document that facet panel geometry is internally derived from figure-level size

### 2b. Size Coordination Strategy (New)

**Question**: Should facet panel geometry be derived from template `Figure_Width` and `Figure_Height`?

**Answer**: **Yes.**

**Why**:
1. `FacetGrid` computes initial figure geometry from panel `height` and `aspect`.
2. Template later applies `fig.set_size_inches(Figure_Width, Figure_Height)`.
3. If these two systems are tuned independently, resulting plots can look stretched or cramped.

**Proposed policy**:
1. Keep template figure size (`Figure_Width`, `Figure_Height`, `Figure_DPI`) as the authoritative user contract.
2. In facet mode, pass target figure size into `histogram()` and derive internal panel geometry from that target size before creating the `FacetGrid`.
3. Use `facet_ncol` + group count to determine `nrow`, then compute:
    - `panel_width = Figure_Width / ncol`
    - `panel_height = Figure_Height / nrow`
    - `facet_height = panel_height`
    - `facet_aspect = panel_width / panel_height`
4. Clamp extreme aspect ratios if needed (for readability and to avoid degenerate layouts).
5. Keep final `fig.set_size_inches()` in template as a consistency snap, not a competing sizing system.

**Action items**:
- [ ] Define minimum safe panel size/aspect guardrails (for dense facet grids)
- [ ] Compute derived facet geometry when `facet=True` using template-provided `target_fig_width`/`target_fig_height` + `facet_ncol`
- [ ] Document precedence and formulas in template docstring/comments

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
- [x] Replace line 723 with: `ax_i.set_title(f'{groups[i]}')`
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

## Decision Log (2026-03-31)

### D1. Facet Parameter Exposure

**Decision**: Keep only essential facet controls public.

**Public/user-facing**:
- `facet` (behavior switch)
- `facet_ncol` (clear, intuitive layout control)
- Existing template-level `Figure_Width`, `Figure_Height`, `Figure_DPI`

**Internal-only**:
- `target_fig_width`, `target_fig_height` (template -> histogram sizing hints)

**Remove from histogram API contract**:
- `facet_vertical_threshold`
- `facet_height`
- `facet_aspect`

**Rationale**:
- Avoid user confusion from overlapping sizing systems.
- Keep template APIs simple and consistent across plotting templates.

### D2. Sizing Precedence Rule

**Decision**: Template figure sizing is authoritative.

In facet mode, final output size should be controlled by template-level
`Figure_Width`/`Figure_Height` (and DPI), not by exposing
panel-level geometry knobs in JSON/template interfaces.

Implementation policy: pass target figure size from template into histogram,
derive internal `facet_height`/`facet_aspect` from figure size and facet
grid shape, then construct FacetGrid.

**Rationale**:
- Maintains cross-plot consistency.
- Avoids two competing sizing knobs controlling the same outcome.
- Produces more predictable and visually balanced facet layouts.

### D3. Kwarg Leakage Guardrail

**Decision**: Add explicit safeguard when wiring facet options through the
template.

If facet-related kwargs are passed into `histogram()`, ensure they do not leak
into non-facet seaborn calls.

**Rationale**:
- Prevents accidental forwarding of facet-only keys to `sns.histplot()` paths.

### D5. No Deprecation Cycle Needed

**Decision**: No deprecation handling is required for removed facet geometry
kwargs because these changes are new and not exposed in the template/user path.

**Rationale**:
- No established external usage to preserve.
- Enables cleaner implementation now.

### D4. Label Strategy Resolution Gate

**Decision**: Resolve axis-label strategy before test update.

Choose one behavior and enforce it consistently:
1. Per-axis labels on each facet axis, or
2. Figure-level `supxlabel`/`supylabel`

Only after this decision should `test_facet_plot` assertions be updated.

### Immediate Next Step (Investigation Complete)

Proceed directly with Phase 2 implementation for template wiring and internal
facet size derivation, while keeping D4 (label strategy) as a tracked decision
for test alignment in Phase 3.

---

## Helper Function Decision

`_compute_global_bin_edges()` remains as a module-level helper in
`src/spac/visualization.py`.

Reasoning:
- Although compact, it captures reusable histogram/facet consistency behavior.
- This pattern is likely to be reused when faceting logic is added to other
    visualizations (e.g., scatter) later.
- No further helper refactoring is needed for this histogram-focused iteration.

---

## Priority Order

1. **HIGH PRIORITY**: 
   - Fix bug at line 723 (broken title setting)
   - Extract `_compute_global_bin_edges()` module-level function
   
2. **MEDIUM PRIORITY**: 
    - Wire `facet`/`facet_ncol` and internal figure-size hints through template
    - Remove unnecessary facet geometry knobs from histogram API contract
   - Plan for axis abbreviation/rotation in template layer
   
3. **LOW PRIORITY**: 
   - Investigate `fig.axes` simplification (test carefully before changing)
   
4. **DEFERRED**: 
   - Update facet tests after implementation finalized

---

## Implementation Checklist

**Phase 1: Bug Fixes and Core Refactoring**
- [x] Fix line 723 bug: restore `ax_i.set_title(f'{groups[i]}')`
- [x] Create `_compute_global_bin_edges()` at module level (line ~400)
- [x] Replace together mode bin computation (lines 657-662) with helper call
- [x] Replace facet mode bin computation (lines 769-774) with helper call
- [ ] Run tests: `python -m pytest tests/test_visualization/test_histogram.py`

**Phase 2: Template Layer Enhancement**
- [ ] Add `facet_ncol` to `histogram_template.py` as the only user-facing facet layout control
- [ ] Add `facet` to `histogram_template.py` as user-facing behavior switch
- [ ] Pass `target_fig_width`/`target_fig_height` into `histogram()` as internal sizing hints
- [ ] Remove `facet_vertical_threshold`/`facet_height`/`facet_aspect` from histogram docstring and accepted public controls
- [ ] Document sizing precedence in template docstring: `Figure_Width`/`Figure_Height` are authoritative
- [ ] Add derived geometry step: compute internal `facet_height`/`facet_aspect` from figure size + facet grid shape
- [ ] Define and document minimum panel-size/aspect guardrails
- [ ] Add safeguard to prevent facet-only kwargs from leaking into non-facet seaborn paths
- [ ] Plan axis abbreviation feature for template (if not already present)

**Phase 3: Testing and Cleanup (Deferred)**
- [ ] Update `test_facet_plot` to match final label strategy
- [ ] Add test for global bin edge consistency across facets
- [ ] Add test for facet layout parameters
- [ ] Consider: test for `fig.axes` simplification if implemented

**Phase 4: Future Unification**
- [ ] Make facet options universal across visualizations later

---

## Notes for Implementation

- Run tests after each change: `python -m pytest tests/test_visualization/test_histogram.py`
- The existing test `test_facet_plot` may fail until Phase 3 (expected)
- Focus on Phase 1 (bug fixes + refactoring) before moving to template enhancements
