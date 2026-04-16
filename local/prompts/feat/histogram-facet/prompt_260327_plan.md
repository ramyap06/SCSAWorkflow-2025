## Background

- Repo: https://github.com/ramyap06/SCSAWorkflow-2025
- Branch: feat/histogram-facet
- Commit: 6b69eb514029e09384f3706ec4a97c09f3f02625
- Enviroment: Github Copilot CLI
- Mode: Agent
- Model: Claude Opus 4.5

## Content

Hi. I have finished my work on this facet plot for histograms. Please check the diff from current local branch "feat/histogram-facet" to "FNL-dev" which is my target branch to understand the work of this facet options. Now I will going to code review. There are a lot of work to do, but before that I think I should finish the following two first:

1. Currently the global_bin_edge for facet is computed in the same way as the together mode above. This is kind of duplicating the code, and according to the CONTRIBUDING.md requirement, it is better to make this modular and reusable. Could you help me refactor the code to do it? Also, I am wondering where should I put the reusable code - as a subfunction or a separate function? You may check the surrounding code and also other functions and decide which way is a better practice

2. The new change adds 4 more input parameters, but maybe some of them are not so appropriate to be there. I would like to introduce another background to you: we are also working on developing a UI in the same time, called SPAC Shiny, and are actually refactoring the way of its function calling using the template method. Please see the attached histogram_template.py - the new function we will use to plot the histogram will be the run_from_json function in this file instead of the "bare" histogram function in the current visualization.py. Then you can see that it allows much more parameters, including figure dpi, width. So my concern is:  

  (a) According to the way previous developers did for this additional layer of run_from_json on the original bare function, do you think people are putting more core functionality in the bare function while expanding the customizable parameters in the run_from_json? 

  (b) Then in light of this, could you figure out a better way of distributing the newly added facet-option functionality across the two layers? should we actually move some of the functionality to the template function instead? 

  (c) Currently we are also adding functionality like axis abbreviation and rotation on the Shiny side. I remember that my mentor said that we'd better expanding this functionality using the template method so that this can be swiftly implemented in other platforms like galaxy. I am not sure if this is what he mean. According to the context and also the general practice in industry, do you think I should move those ui functionality from the Shiny frontend to this run_from_json backend?

3. I am also concerned about the way of using axes, axs, ax_array, ax there. I think the point is that users may pass their own ax to the function. However, do you think we can simplify some of the newly added code of computing these axes stuffs? I doubt that the ax_array is redundant and maybe we can use fig.axes instead to get it directly? Please investigate to tell me if this is the case.

These stuffs are the most concerned ones I have now. Lets deal with these first. There are a lot more need to do before merging but let's postpone them later.

Please only draft a plan and do not implement it for now. I will investigate and decide whether to implement it later.
