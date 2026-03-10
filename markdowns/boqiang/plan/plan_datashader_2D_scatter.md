# Plan: Datashader Heatmap for 2D Scatter

## TL;DR
Create a new standalone function `visualize_2D_scatter_datashader` with the same
signature and return contract as `visualize_2D_scatter`, but rendering density
heatmaps via datashader instead of per-point `plt.scatter`. Handles three label
cases (none, continuous, categorical) with mathematically appropriate aggregation
for each.  Default overlay for categorical; optional facet mode.

---

## Phase 1 — New function skeleton & no-label case

1. Define `visualize_2D_scatter_datashader(x, y, labels=None, theme=None,
   ax=None, x_axis_title=..., y_axis_title=..., plot_title=None,
   color_representation=None, facet=False, **kwargs)` in
   `src/spac/visualization.py`, placed right after `visualize_2D_scatter`.
   - Same return: `(fig, ax)`.
   - `facet` param only used when labels are categorical.

2. Input validation — reuse same checks as `visualize_2D_scatter`:
   iterable check, length match, labels length match.

3. Figure/ax creation — mirror current logic:
   - `fig_width = kwargs.get('fig_width', 10)`
   - `fig_height = kwargs.get('fig_height', 8)`
   - If `ax is None`, create `fig, ax = plt.subplots(...)`.
   - Canvas pixel resolution = `int(fig_width * 100)` x `int(fig_height * 100)`.

4. **No-label case** (`labels is None`):
   - Build `pd.DataFrame({"x": x, "y": y})`.
   - Create `ds.Canvas(plot_width=px_w, plot_height=px_h, x_range, y_range)`.
   - Aggregate: `canvas.points(df, "x", "y", agg=ds.count())`.
   - Shade: `tf.shade(agg, cmap=<matplotlib cmap name>)`.
   - Convert to PIL, display via `ax.imshow(img, origin='lower', extent=...)`.
   - Add colorbar showing point-density scale.
   - Mathematically: each pixel = count of points falling in that pixel bin.

5. Set axis labels, title, aspect ratio, return `(fig, ax)`.

**Verification**: Unit test with random x, y (no labels), assert returns
`(Figure, Axes)`, assert `ax` contains an `AxesImage`.

---

## Phase 2 — Continuous label case

6. **Continuous labels** (`labels is not None` and NOT categorical):
   - *depends on step 4*
   - Aggregate: `canvas.points(df, "x", "y", agg=ds.mean("value"))` where
     `df["value"] = labels`. This computes mean label value per pixel,
     analogous to the colorbar scatter in current `visualize_2D_scatter`.
   - Shade: `tf.shade(agg, cmap=<theme>, how='linear')`.
   - Display via `ax.imshow(...)`.
   - Add colorbar with label `color_representation`.
   - Mathematically: each pixel = mean feature intensity of points in that bin.
     This is the datashader equivalent of `ax.scatter(x, y, c=labels, cmap=...)`.

**Verification**: Unit test with random x,y and continuous labels array,
assert colorbar present, assert returns `(Figure, Axes)`.

---

## Phase 3 — Categorical label case (overlay mode, default)

7. **Categorical labels, overlay** (`facet=False`, default):
   - *depends on step 4*
   - Convert labels column to `pd.Categorical`.
   - Use `canvas.points(df, "x", "y", agg=ds.count_cat("labels"))` to get
     per-category counts in each pixel.
   - Shade: `tf.shade(agg, color_key=<dict mapping category->color>)`.
     Build `color_key` from the same tab20/tab20b/tab20c palette used by
     `visualize_2D_scatter`.
   - Display via `ax.imshow(...)`.
   - Build a manual legend matching category names to colors (using
     `matplotlib.patches.Patch`), placed at `bbox_to_anchor=(1.25, 1)` to match
     current legend placement.
   - Mathematically: each pixel is colored by whichever category has the most
     points in that bin (datashader's default categorical shading), with alpha
     blending where categories overlap.

**Verification**: Unit test with categorical labels, assert legend present,
assert `AxesImage` in ax.

---

## Phase 4 — Categorical label case (facet mode, optional)

8. **Categorical labels, facet** (`facet=True`):
   - *depends on step 7*
   - Create subplots grid: `n_cats` panels (use `math.ceil(n_cats/3)` rows, 3 cols).
   - For each category, filter df, aggregate `ds.count()`, shade, imshow into
     its own subplot. All subplots share same x_range/y_range.
   - Return `(fig, axes[0])` for API compat, or `(fig, axes)` — TBD, but
     recommend `(fig, axes[0])` to keep caller contract.
   - Delete unused subplot axes.

**Verification**: Unit test with categorical labels + `facet=True`, assert
correct number of subplots.

---

## Phase 5 — Wire into callers (no code changes needed if separate function)

9. **dimensionality_reduction_plot**: No change needed now. Callers can switch
   to calling `visualize_2D_scatter_datashader` directly when desired.
   Later, can add a `use_datashader=False` param to `dimensionality_reduction_plot`
   that dispatches to the new function.

10. **Shiny scatterplot_server.py**: No change needed now. Later, add a UI
    toggle (e.g. checkbox "Use heatmap mode") that calls
    `spac.visualization.visualize_2D_scatter_datashader(...)` instead.

---

## Relevant Files

- `src/spac/visualization.py` — add new function after `visualize_2D_scatter` (~line 463).
  Reference `visualize_2D_scatter` for signature, validation, theme dict, figure creation.
  Reuse `ListedColormap`, tab20 palette logic for categorical colors.
- `tests/test_visualization/` — add `test_visualize_2D_scatter_datashader.py`.
  Model after `test_visualize_2D_scatter.py` structure.
- `environment.yml` line 20 — `datashader` already listed as dependency.
- `setup.py` line 38 — `datashader` already in install_requires.

---

## Verification

1. Run `python -m pytest tests/test_visualization/test_visualize_2D_scatter_datashader.py -v`
2. Manual smoke test: call function with ~5000 random points, no labels → density image.
3. Manual smoke test: call with continuous labels → colored density with colorbar.
4. Manual smoke test: call with 3-category labels, facet=False → overlay with legend.
5. Manual smoke test: call with 3-category labels, facet=True → 3-panel faceted plot.
6. Confirm existing `test_visualize_2D_scatter.py` still passes (no regressions).

---

## Decisions

- **Separate function** (not a mode inside `visualize_2D_scatter`), to keep existing
  function stable and tests unaffected.
- **Default overlay** for categorical labels; facet available via `facet=True`.
- **Canvas resolution auto-calculated** from figure size (width*100, height*100 pixels).
- **Aggregation strategy per case**:
  - No labels: `ds.count()` → point density.
  - Continuous: `ds.mean("value")` → mean feature value per pixel.
  - Categorical: `ds.count_cat("labels")` → per-category density with color blending.
- **Scope exclusions**: `spac_datashader_labeled_heatmap` and `heatmap_datashader` left
  as-is (can be removed in a separate cleanup PR). No Shiny UI changes in this PR.
