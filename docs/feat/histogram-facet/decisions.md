# Decisions

### D1. Facet Parameter Exposure (Task 2)
Decision:
Expose only facet and facet_ncol as user-facing facet controls.

Details:
Figure_Width/Figure_Height/Figure_DPI remain template contract; target_fig_width/target_fig_height stay internal hints.

Rationale:
Keep user API simple and avoid competing layout knobs.

### D2. Sizing Precedence Rule (Task 2)
Decision:
Template figure sizing is authoritative.

Details:
Histogram derives internal facet panel geometry from target figure size and facet grid shape.

Rationale:
Predictable output and cleaner cross-template behavior.

### D3. Kwarg Leakage Guardrail (Task 4)
Decision:
Facet layout kwargs must not leak into non-facet seaborn calls.

Details:
Layout hints are parsed and removed before histplot paths.

Rationale:
Prevent accidental runtime errors and behavior drift.

### D4. Label Strategy (Task 6)
Decision:
Use figure-level labels in facet mode.

Details:
Facet axes clear per-axis labels; figure sets supxlabel/supylabel.

Rationale:
Cleaner multi-panel readability and consistent presentation.

### D5. No Deprecation Cycle Needed (Task 2)
Decision:
No deprecation policy needed for removed facet geometry kwargs in user API.

Details:
Changes are new and not part of long-standing external template contract.

Rationale:
Keep implementation clean without migration overhead.

### D6. Bins Policy Scope (Task 5)
Decision:
Template owns user-facing bins policy; bare histogram keeps internal fallback normalization.

Details:
Do not perform broad legacy refactor; keep targeted default-like bins fallback and facet shared-bin consistency.

Rationale:
Balance stability with performance and consistency.

### D7. Helper Function Placement (Task 1)
Decision:
Adopt mixed helper boundaries for facet scope; naming/boundary refactor proceeds under Task 10.

Details:
- Keep `_parse_histogram_layout_kwargs` and `_derive_facet_geometry` at module level until Task 10 refactor.
- In Task 10, rename `_parse_histogram_layout_kwargs` to a more universal helper name.
- Move `compute_global_bin_edges` and `resolve_hist_axis_labels` into `histogram` scope.
- Keep `cal_bin_num` and `calculate_histogram` nested and unchanged.

Rationale:
Preserve reuse for facet-layout logic while minimizing histogram-only API surface and avoiding unnecessary renaming churn.

### D8. Testing Scope and Granularity (Notes for Implementation, Task TBD)
Decision:
Use a hybrid test strategy: expand existing tests where helpful, split high-friction facet cases, keep a small cross-mode consistency matrix, and keep template tests I/O-oriented.

Details:
- Keep old tests and expand selected assertions when behavior changed.
- Split the current facet test into focused structure/title/label checks if that improves readability.
- Use direct helper tests only for helpers with meaningful branching; otherwise cover via public histogram behavior.
- Keep template tests primarily focused on I/O artifacts and wiring.

Rationale:
This satisfies CONTRIBUTING.md coverage expectations without over-testing internals or duplicating behavior across too many files.

### D9. Coverage Policy for New Facet/Template Tests (Notes for Implementation, Tasks TBD)
Decision:
Write targeted regression tests for the new facet contracts, bins fallback behavior, and template facet parameter wiring.

Details:
- Keep meaningful label assertions.
- Avoid redundant or over-constraining rendering assertions.
- Validate new kwargs and template outputs at the level of the public contract.

Rationale:
Protect the actual user-visible behavior while keeping the suite maintainable.

### D10. Facet `ax` + Lifecycle Refactor Scope (Task 9)
Decision:
For this PR, ban `facet=True` with external `ax`, refactor figure lifecycle handling, and keep existing facet internals unchanged.

Details:
Apply a targeted change only: explicit validation for unsupported external-`ax` facet usage, plus ownership-safe figure creation/closure cleanup.

Rationale:
Addresses notebook/lifecycle issues without broad internal redesign.

### D11. Facet Bin/Bar/Label Test Scope (Task 7)
Decision:
Keep bar-level assertions lightweight across Task 12 split/smoke tests and Task 7 follow-up coverage, place shared-bin coverage under Task 1, and keep default-like bins fallback coverage under Task 5.

Details:
- Cover facet-specific bin contracts directly while avoiding brittle rendering-detail assertions.
- Split coverage by concern: shared-bin contracts in Task 1, default-like bins fallback in Task 5, label contracts in Task 6, and remaining histogram/facet checks in Task 7.

Rationale:
- Protects key facet behavior with stable, maintainable tests.
- Reduces Task 7 scope while keeping ownership of each test group clear.

### D12. Template Test Scope Restriction (Task 8)
Decision:
Template histogram tests remain I/O-focused for this PR.

Details:
Avoid template-side logic tests; keep only output-contract coverage. Non-I/O template assertions are out of scope for this PR.

Rationale:
Aligns with repository template-test style and avoids duplicating visualization logic checks.

### D13. Facet Geometry Ratio Policy (Task 4)
Decision:
Keep current `_derive_facet_geometry` ratio approximation for this PR.

Details:
Do not adjust the geometry model for suptitle/suplabel effects in this PR.

Rationale:
Minimize layout regression risk and keep scope focused on facet correctness and tests.

### D14. Facet Figure Title Ownership (Task 8)
Decision:
Do not add figure-level suptitle in `histogram`; keep figure title responsibility in template/post-processing layer.

Details:
`histogram` keeps plotting semantics and axis/super-label behavior; template controls user-facing figure title composition.

Rationale:
Preserves layer boundaries, keeps facet-mode scope focused, and matches current template-first user-facing workflow.

### D15. Facet X-Label Ownership and Duplication Rule (Task 8)
Decision:
For facet mode, keep x-label semantic content in `histogram`; template may adjust label presentation only.

Details:
Avoid recomputing/reassigning facet x-label text in template when it already comes from `histogram`; any presentation-only adjustments should not duplicate semantic label assignment.

Rationale:
Prevents duplicated labeling logic and reduces drift between plotting core and template post-processing.

### D16. Facet Annotation/Categorical Test Depth Resolution (Task 7)
Decision:
Use a mixture approach: keep lightweight facet bar assertions, and add one focused annotation-based categorical facet regression test.

Details:
Avoid strict bar center/height rendering assertions for facet plots; ensure annotation/categorical facet behavior is covered with stable, contract-level checks.

Rationale:
Closes the annotation/categorical coverage gap while keeping facet tests robust and maintainable.

### D17. Facet Label-Issue Resolution Path (Task 11)
Decision:
Solve the facet long x-label overlap issue through geometry-aware facet sizing in the core plotting path, with template-level hint passing only.

Details:
For this issue, the chosen path is:
- derive facet geometry from tick-label burden and rotation when explicit figure-size hints are absent,
- keep the template as a hint-passing layer,
- defer axis-label abbreviation to future work outside this PR.

Rationale:
Preserves label text while fixing the core readability problem caused by an under-sized plot area.

### D18. Facet Test Split Task Boundary (Task 12)
Decision:
Track facet-test decomposition as dedicated Task 12 and remove overlapping split/smoke scope from Task 7.

Details:
- Task 6 owns figure-level label policy checks and non-default stat label mapping.
- Task 12 owns splitting the heavy facet test and defining one smoke-path facet test.
- Task 7 owns remaining histogram/facet coverage beyond the split work.

Rationale:
Clarifies ownership, avoids duplicated action items, and keeps facet-test changes reviewable.

### D19. Histogram Helper Locality and Docstring Style (Task 10)
Decision:
Keep histogram-only helpers local to `histogram` and use compact NumPy-style docstrings.

Details:
- Keep `_derive_facet_geometry` as the module-level layout helper; `histogram` pops facet layout hints directly.
- Keep `compute_global_bin_edges` / `resolve_hist_axis_labels` local inside `histogram`.
- Use concise NumPy-style Parameters/Returns sections for the two local helpers.

Rationale:
Preserves locality for histogram-only logic, reduces module-level noise, and keeps documentation consistent with CONTRIBUTING.md.

### D20. Facet `facet_ncol` Test Contract Priority (Task 4/7)
Decision:
Prioritize tests for documented `facet_ncol` inputs (`"auto"` and positive int); keep invalid-input fallback checks lightweight.

Details:
Removed non-essential float-like (`2.0`) `facet_ncol` unittest coverage and added explicit `"auto"` layout assertion.

Rationale:
Aligns coverage with user-facing contract in the histogram docstring while retaining one guardrail for permissive fallback behavior.

### D21. Helper-Level Unit-Test Scope Tracking (Task 13)
Decision:
Track independent `_derive_facet_geometry` unittest work as a separate remaining task (Task 13), not as a completed part of Task 4.

Details:
Task 4 remains completed for feature-level facet behavior/tests, while helper-level dedicated unittest coverage is explicitly deferred and tracked for a later implementation pass.

Rationale:
Keeps completion status truthful and aligns with CONTRIBUTING guidance that helper functions should have dedicated unit tests.

### D22. Facet Hint Validation Ownership Simplification (Task 15)
Decision:
Remove the generic `normalize_positive_number` path from facet layout hint handling and use explicit local parsing in `histogram`.

Details:
- Keep template as the strict user-facing boundary for `Facet_Ncol` and figure-size inputs.
- Keep `histogram` minimally defensive for direct calls with explicit local parsing/validation of facet layout hints.
- Keep `_derive_facet_geometry` pre-normalized and geometry-only.

Rationale:
The facet hint contract is small and plot-specific, so explicit parsing is clearer than introducing a reusable normalization helper that only obscures ownership.

### D23. Facet Auto Figure Size and Long-Label Geometry Policy (Tasks 2, 11)
Decision:
Allow facet template figure size to remain `"auto"` and let histogram derive default geometry from tick-label burden when explicit size hints are absent.

Details:
- In facet mode, `"auto"` width/height is passed as `None` hints instead of forcing a fixed figure size.
- Explicit figure-size hints remain authoritative.
- The default long-label heuristic should primarily add panel height and only slightly tighten aspect.

Rationale:
This keeps the user-facing template simple while improving default readability for dense categorical facet plots.

### D24. Categorical `bins`-Ignore Regression Scope (Task 7, resolves former Issue 2)
Decision:
Keep the categorical facet `bins`-ignore regression lightweight.

Details:
- Reuse `self.adata` where practical.
- Compare one representative facet axis for shared categorical centers/ticklabels instead of duplicating the same assertion across every panel.

Rationale:
This preserves the user-visible contract without over-weighting the suite with repetitive rendering checks.

### D25. Promote Former Issues 2-3 Into Planned Tasks (Tasks 16 and 18)
Decision:
Convert the remaining facet title-layout bug and histogram `df`-contract inconsistency into dedicated planned tasks.

Details:
- Former Issue 2 becomes Task 16 so title/layout handling has a concrete implementation path at the template/core boundary.
- Former Issue 3 becomes Task 18 so histogram return-data consistency can be handled as an explicit contract task rather than a vague open issue.

Rationale:
Both concerns now have clear implementation directions and are better tracked as executable tasks than as open-ended issues.

### D26. Defer Helper Relocation to `spac.utils` (Former Open Issue 2)
Decision:
Defer any relocation of histogram-local helpers into `spac.utils` to future work rather than this facet PR.

Details:
- Current histogram-local helpers remain close to the plotting behavior they support.
- Moving them now would expand public/internal surface and likely require extra direct unittest coverage without clear near-term reuse.

Rationale:
Keep this PR focused on facet correctness and review readiness; broader helper-surface cleanup is better handled in a later refactor.

### D27. Add Thin Numeric-Annotation Facet Smoke Coverage
Decision:
Add one lightweight numeric-annotation facet smoke unittest in this PR.

Details:
- Treat this as boundary coverage for annotation-sourced numeric input rather than a deep rendering test.
- Reuse existing facet smoke-test style and avoid duplicating label/geometry assertions already covered elsewhere.

Rationale:
Numeric annotation is user-exposed, so one thin smoke test is worth keeping even though it shares most downstream logic with numeric feature facet mode.

### D28. Promote Numeric/Continuous Facet `group_by` Rejection Into a Task
Decision:
Handle numeric/continuous `group_by` rejection in facet mode as an in-scope validation task rather than future work.

Details:
- Continuous numeric grouping can generate one facet per distinct value under the current implementation.
- For this PR, facet `group_by` is treated as a discrete grouping label.

Rationale:
This is important facet-mode validation and should be fixed in the current workstream rather than deferred.

### D29. Seaborn Passthrough-Kwarg Validation/Test Scope
Decision:
Do not add dedicated unittests or extra custom validation for seaborn passthrough kwargs such as `shrink` and `alpha` in this PR.

Details:
- `histogram` largely forwards these kwargs without adding SPAC-specific semantics.
- If passthrough kwargs later need special handling, they can be revisited in a focused follow-up.

Rationale:
Avoid spending review scope on forwarded seaborn behavior that SPAC does not reinterpret.

### D30. Promote Plotting-Control Validation Cleanup Into Task 20
Decision:
Handle plotting-control validation cleanup as an explicit task instead of leaving it as an open scope question.

Details:
- Remove template-side explicit allow-list validation for `multiple`, `element`, and `stat` unless SPAC is intentionally defining a narrower contract than seaborn.
- Replace the template `together=False` plus `multiple` coercion with a clear `ValueError`.
- Add the same minimal `together=False` plus `multiple` conflict validation in `histogram()` because it is also a public callable function.

Rationale:
This separates third-party passthrough validation from SPAC-owned semantic validation and gives the remaining review work a concrete executable scope.

### D31. Replace Facet Numeric-Only `group_by` Rejection With Group-Count Guardrail
Decision:
Use grouped-mode maximum-group-count validation as the primary `group_by` safety contract instead of facet-only numeric/continuous rejection.

Details:
- Apply the guardrail uniformly across grouped plotting modes (facet/together/separate grouped paths).
- Default threshold is `max_groups=20`, with explicit user override support.
- `max_groups=None` disables the cap intentionally.
- On threshold violation, raise one clear `ValueError` message (no extra warning) including observed count, threshold, and override guidance.

Rationale:
Cardinality is the direct source of pathological grouped plots; this contract is more robust and user-actionable than dtype-only rejection.

### D32. Track Template Validation Convention as Dedicated Task
Decision:
Create Task 21 to lock histogram-template validation conventions before finishing Task 17 guardrail implementation.

Details:
- Do not assume globally string-only inputs across template params.
- Use explicit token normalization for token-fields, typed-first handling for typed fields.
- Use Task 21 outcome as prerequisite for final Task 17 `Max_Groups` contract implementation.

Rationale:
This prevents subtle regressions (for example, `Group_by` `"None"` vs `None`) and keeps Task 17 focused and testable.

### D33. Lock Current `Max_Groups` Contract for Task 17 Completion
Decision:
Complete Task 17 with a concise contract: positive integer threshold or `"unlimited"` bypass, with `None` resolving to default threshold behavior.

Details:
- Template now normalizes `Group_by` token values via `text_to_value(...)` to avoid `"None"` vs `None` ambiguity.
- Template and core both pass through/accept `"unlimited"` for explicit cap disablement.
- Default behavior remains `max_groups=20` when parameter is omitted (and explicit `None` is treated as default threshold behavior in current core parsing).
- Focused histogram tests were simplified to behavior-level checks rather than exhaustive parser-token branching.

Rationale:
This preserves a strict, understandable user-facing guardrail while keeping direct-core validation concise and test maintenance lightweight.

### D34. Track `_parse_optional_number` Design as an Open Task-21 Issue
Decision:
Keep `_parse_optional_number` simplification/revision as an explicit open issue under Task 21 rather than quietly refactoring it during unrelated task execution.

Details:
- The helper was introduced recently and may currently be broader than needed for active histogram hint parsing.
- The follow-up should lock one clear direction: retain with a narrower contract, simplify call patterns, or replace with explicit local parsing.
- Preserve current behavior contracts while reducing confusion and redundant abstraction.

Rationale:
This keeps review scope controlled and makes the helper-boundary decision explicit and traceable before additional cleanup work proceeds.

### D35. Complete Task 16 With Template-Side Row-Scaled Layout Handling
Decision:
Close Task 16 using a concise template-side facet layout rule that scales `tight_layout` spacing by detected facet-row count.

Details:
- Keep facet panel geometry ownership in `histogram` core and use template only for final figure-level spacing polish.
- Keep one simple row-factor algorithm in template (instead of split compact/non-compact branches) so 1/2/3/4+ row grids can reserve different title/label room.
- Keep this as a pragmatic layout policy for the current PR; deeper geometry-model changes remain future work.

Rationale:
This resolves title overlap in dense facet grids while keeping template logic short and maintainable.

### D36. Narrow `_parse_optional_number` to Shared Numeric Parsing
Decision:
Keep `_parse_optional_number` in `histogram()` but narrow it to shared numeric parsing mechanics only, with special-string policy remaining explicit at each call site.

Details:
- The helper now handles only `None` defaulting, explicit token lookup, numeric coercion, finite checks, and positive-value checks.
- Special contracts remain visible where they matter:
  - `max_groups` explicitly maps `"unlimited"`,
  - `facet_ncol` explicitly maps `""`, `"auto"`, and `"none"`,
  - other facet hints use plain numeric parsing.
- Error messaging is unified through the helper so the refactor reduces repeated code instead of adding more branches.

Rationale:
This keeps the overall diff smaller and easier to review than either broad generic parsing or repeated local validation blocks.

### D37. Close Task 21 at Histogram-PR Scope
Decision:
Complete Task 21 once the histogram-specific template audit is finished and the newly introduced validation/helper behavior is aligned, without expanding into repo-wide template convention cleanup.

Details:
- Review against `upstream/dev` confirmed that mixed token/typed template parsing is a broader repository pattern, not a histogram-only divergence.
- For this PR, the important requirement is that the newly introduced histogram-specific changes do not create misleading validation behavior.
- Keep strict token contracts where they intentionally fail fast, and defer broader boolean/string normalization policy to future work outside this PR.

Rationale:
This keeps the histogram facet PR focused on its own boundary changes instead of turning it into a repository-wide template cleanup.

### D38. Ignore `multiple` Outside Same-Axes Grouped Overlays
Decision:
Treat `multiple` as meaningful only for grouped overlays on a shared axes, and ignore it in grouped-separate and facet grouped paths.

Details:
- Template no longer coerces `multiple` to `"dodge"` when `Together=False`.
- Template forwards `multiple` only when `group_by` is set and `together=True`.
- `histogram()` defensively drops `multiple` in grouped non-overlay paths when `together=False`.
- Template no longer keeps a SPAC-side allow-list for seaborn-native `multiple`, `element`, and `stat` values.

Rationale:
This prevents frontend-driven crashes like `multiple="fill"` in separate-group rendering without inventing fake semantics for a parameter that has no real job in those branches.

### D39. Reuse Grouped Histogram-Bin Table Logic for Task 18
Decision:
Implement Task 18 by reusing one grouped histogram-bin table builder across grouped overlay and facet paths, while narrowing the public contract change to facet return data only.

Details:
- Extract one histogram-local grouped-table helper from the existing `together=True` branch.
- Reuse the helper in facet mode so returned facet `df` matches grouped histogram-bin structure with `group_by` metadata.
- Keep non-facet return contracts unchanged in this task even though the implementation path is shared.

Rationale:
This keeps Task 18 small, avoids duplicated grouped-bin logic, and limits user-visible return-data change to the intended facet branch.

### D40. Close Task 18 by Matching Existing Count-Table Pattern
Decision:
Finish Task 18 by making facet follow the same precomputed histogram count-table pattern already used by the single-plot and `together=True` branches.

Details:
- Facet should plot from precomputed grouped histogram-bin data and return that same table as `df`.
- Shared grouped-table construction should absorb shared-bin derivation instead of leaving facet with a separate raw-data plotting path.
- Broader seaborn-fidelity questions (for example, KDE/raw-observation semantics) are explicitly deferred.

Rationale:
Branch consistency is the right scope for this PR, while full raw-data/seaborn-fidelity redesign would expand review scope too far.

### D41. Keep Categorical Shared Slots Internal and Rice Fallback Numeric-Only
Decision:
Finalize grouped shared-bin cleanup by keeping categorical shared-slot handling internal to the grouped histogram-table helper and limiting Rice-rule auto-bin fallback to numeric data only.

Details:
- `build_grouped_histogram_table` now owns grouped shared-bin/category derivation for the `together=True` and facet precomputed paths.
- Numeric grouped plotting still reuses shared numeric bin edges outward for seaborn plotting.
- Categorical grouped/facet paths keep shared category padding inside `hist_data` rather than pushing category state through `kwargs["bins"]`.
- Default-like `bins` values no longer trigger meaningless Rice-rule computation for categorical data.

Rationale:
This keeps grouped/facet plotting consistent while reducing redundant `bins` handling and avoiding misleading categorical auto-bin behavior.
