All documented in the plan.md (4/15/26)

Previous prompt (4/14/26)

OK. 

1. Some general ideas:

(a) Let's not bother with the unit tests for previous contribution for other mode, but only on facet mode that I (and the previous developer who initially write facet) contribute. 

(b) After reading contributing.md, I find that we must write tests for all our new code, new functions, as well as update some consistency tests across mode. 

(c) Before that, I have a general question: should we keep all original unit tests, or we can expand them to incorporate more assertions?

2. Before writing tests, I find some issues that we might need to re-develop first:

(a) We need to deal with the case that the user pass their own `ax` and turn `facet` mode on. What's your opinion? Do we simply ban this pattern? Because I realize that we actually do not allow this: we always create a new figure for facet mode. If we want to use user `ax`, then we need to deal with this case specifically. But I find that the template actually simply avoid such case, so maybe it's better we ban this?

(b) I remember you mention some figure closing issue. What is that? Do you mean we should change how we open and close figures? Like the newly introduced `created_internal_fig`. I find that this does not align with the test logic `test_ax_passed_as_argument`, but the test is for old, not group_by logic. What's your opinion?

(c) Can you re-check if my plotting logic in facet mode correct? I realize that they use a `calculate_histogram` function to compute histogram. Is that necessary? Is that and `sns.histplot` the same as my `sns.FacetGrid` and `map_dataframe` + `sns.histplot`?

(d) Are `tick_params` in line 901-904 necessary?

3. Now for existing tests for facet written by previous developer: I think maybe we should keep these three together, because I find that some other unittests in the file also test more than one feature in a single test. They are more case-oriented: test if different args passed will produce expected output. What's your opinion? 

4. More tests should be written for facet as we must have unittests according to the contributing doc for each function and file. Here are my proposal:

(a) We should test when `'auto'` or `None` like value is passed to `bins`, whether the output reflects the rice-rule. There is a test doing this (`test_default_bins_calculation`), so maybe expand this test a bit more? 

Besides, do you think we should also test if `facet` is turned on, this binning is still as expected? So then we shall also expand this test to include facet mode.

(b) We should test the new input validation for facet in line 733-746 in `visualization.py` - make sure group_by is not `None` and together is not `True`

(c) We should test that in the new facet mode, we get the desired `facet_ncol` in the end, and if we pass `facet_fig_width`, and `facet_fig_height` in kwargs, the final output fig reflect it correctly. We should test different input cases for `facet_ncol`, including those invalid cases and test if type check is working (like in `test_negative_values_x_log_scale`)

(d) We should test if axis-level and fig-level title and labels are as expected in facet mode. This is also the one that currently fails. Some examples are `test_tile`, `test_y_log_scale_axis`, etc. See later discussions on the helper function `_resolve_histogram_axis_label`.

Also can you explain the `"{col_name}"` in line 912 to me? `col_name` is not a pre-defined variable, and we are not using `f"{...}"`. What's this col_name? Interestingly it passed the unit-test written by the previous developer.

(e) To make sure the facet plotting are as expected, I think we also need to do simple examples as in `test_layer`. Current `test_facet_plot` only do feature, and we may need an `annotation`. We should also check bar height, bar centers are expected. Check return type and shape, like in `test_histogram_feature`, `test_histogram_annotation`, etc.. 

But wait: I find that previous tests only check bars in `test_histogram_feature`, but not in later cases. Does that mean I do not need to check bars in facet? Emm maybe still need, I am not sure. What's your opinion?

5. For helper functions, we must also have unittests according to the contributing doc for each function and file. It seems that each function in the `visualization.py` has an independent test file, including at least some of the existing helper functions. 

But before that, maybe we can decide on whether to move some of the helper function inside the histogram function which are only for the histogram, rather than can be applied universally. Or maybe not - what's your opinion?

(a) For example, should we rename the `_parse_histogram_layout_kwargs` helper more universally, because it is actually filtering and normalizing facet params, not specifically for the `histogram` function? Or should we extract the filtering logic out, and combine the rest of the validation to the `_derive_facet_geometry` helper function below because it is only used by the facet plotting? 

Then should we also write a unit test file for it? You may inspect other functions as a reference.

(b) We do have to write unittests for `_derive_facet_geometry`, and we need to deal with different passing args. 

Now I also have a question for this helper function. I realize that the current logic is to use `fig_width/height` to determin `facet_height/aspect`. However, if we have suptitles, suplabels, this will destroy the ratio, isn't it? I am not sure. Should we leave it? Because it is not a big deal?

(c) Should we move the `_resolve_histogram_axis_label` inside the histogram function? Or keep it outside? What's your opinion? I find that actually there are numerous label checks in the current tests, like `test_y_log_scale_label`, `test_group_by_together_with_y_log_scale`, etc., but some of them are focused on labels while some others deal with other checks in the same time. So combine to `histogram` and expand unit-tests for label checks, or keep it separated from `histogram`, and create a separate unit-test file and move all current tests related to labels in histogram test to this new unit-test file - what's your opinion? 

Also as you said, we should check ylabels for more stat input since we allow more (proportion, percent). Looks like they have `test_y_axis_label_based_on_stat` and we need to expand it.

Also: in facet mode I write my logic as figure-level, not axis-level. This is approved by my mentor. If we want to test the output histogram's axis label, we need to be careful when we expand the logic. If we only test the helper function then no worries.

(d) For this `_compute_global_bin_edges`, it is more related histogram, and a very small logic. Do you think we should move this back to histogram function? (can it be applied to other visualization?). Otherwise we need to create a new file to write unittests for it. Looks weird -> so think carefully whether we need to keep it separated. 

Also currently we don't have a unit test (looks like no) for it. Is there anything that deserves to test? Maybe we can try to see if bin_edges are consistent across axes? 

6. Could you also read through the test files, and our contributions, and see if there are other cases that need to be written in unittests?

7. We also need to test the newly introduced args: `shrink`, `alpha`

8. For template test, I find that they all validates I/O behaviour only. Seems that we don't need to do the value check. What do you think? 