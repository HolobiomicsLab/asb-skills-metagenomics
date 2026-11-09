---
name: u-curve-visualization
description: Use when progressive permutation results (bin_permute_all output) are
  available and you need to visualize significant-feature counts across mixing proportions
  to assess overall microbiome–group association.
when_to_use_negative:
- When no progressive permutation results exist yet — plot_bin_psig requires the alloutputs
  from bin_permute_all, not raw count data
- When the goal is per-feature robustness ranking (fragility index, progressive coverage)
  rather than overall association — use bin_fragility or bin_progresscoverage instead
- When a formal inferential measure is required — the U-curve is descriptive and the
  method does not control FDR or compute adjusted p-values
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: ProgPermute
  role: R package providing plot_bin_psig and plot_bin_permute_sigcurve for rendering
    the U-curve from bin_permute_all results
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: execution environment for the ProgPermute plotting functions and EPS figure
    export
- name: R-shiny
  role: web interface exposing U-curve visualization without local installation
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:edb44b8616d0629ac8f180abac04ee1cf058713401842772cf7c154f54a4bd1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# u-curve-visualization

## Summary

Renders the ProgPerm U-curve: the number of significant microbial features versus the proportion of label mixing under progressive permutation, where the U-shape conveys the strength of the overall association between the microbiome and the grouping factor.

## When to use

Use when progressive permutation results (bin_permute_all output) are available and you need to visualize significant-feature counts across mixing proportions to assess overall microbiome–group association.

## When NOT to use

- When no progressive permutation results exist yet — plot_bin_psig requires the alloutputs from bin_permute_all, not raw count data
- When the goal is per-feature robustness ranking (fragility index, progressive coverage) rather than overall association — use bin_fragility or bin_progresscoverage instead
- When a formal inferential measure is required — the U-curve is descriptive and the method does not control FDR or compute adjusted p-values

## Inputs

- alloutputs object returned by bin_permute_all (per-scenario differential testing results on testdata1 with grouping factor 'variable' and 267 microbial features)
- psigyrange for the y-axis (e.g. c(0,170))
- samsize (total number of samples/features, e.g. 267) for the scaled sigcurve variant

## Outputs

- EPS file of the U-curve of significant-feature counts versus proportion of mixing (e.g. sigfeatures.eps)
- Scaled U-curve of the proportion of significant features (e.g. bin_sigcurve.eps)
- Location handle (e.g. sigloc) for the rendered plot

## How to apply

After running bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), summarize the per-scenario median p-values into counts of significant taxa nsig(k) at each progressive permutation scenario. Call plot_bin_psig with alloutputs=results and an appropriate psigyrange (e.g. c(0,170) for testdata1) to plot number of significant features versus proportion of mixing, saving to an EPS file such as sigfeatures.eps. Optionally call plot_bin_permute_sigcurve with samsize=267 to render the scaled U-curve of the proportion of significant features. Interpretation: the curve decreases from the observed data (k=0) toward the full permutation scenario k=K_f, forming a U-curve whose shape conveys the strength of the overall association; a near-flat curve is expected only if no group signal exists. Note the U-curve can also be generated with DESeq p-values substituted for the default Wilcoxon rank sum test.

## Related tools

- **ProgPermute** (R package providing plot_bin_psig and plot_bin_permute_sigcurve for rendering the U-curve from bin_permute_all results) — https://github.com/LyonsZhang/ProgPermute
- **R** (execution environment for the ProgPermute plotting functions and EPS figure export)
- **R-shiny** (web interface exposing U-curve visualization without local installation)

## Examples

```
sigloc<-plot_bin_psig(alloutputs=results,psigtitle=NULL,psigyrange=c(0,170),savepsigfile="sigfeatures.eps",psigpicdim=c(10,7))
```

## Evaluation signals

- sigfeatures.eps exists and is a valid EPS figure
- Curve is decreasing from the observed data (k=0) toward the full permutation scenario k=K_f, then U-shaped — comparable to the deposited UCurve.png reference
- y-axis range respects the requested psigyrange (e.g. counts span roughly 0–170 for testdata1)
- A near-flat curve is only expected when no group signal exists; a flat curve on signal-bearing data indicates a pipeline error
- top_pm parameter must be less than the number of microbial features (e.g. 267 < 267 features is the boundary used in the example)

## Limitations

- The U-curve (and AOI/AUMC measures) is descriptive, not inferential — no formal inference method is provided
- The method does not control the false discovery rate or calculate adjusted p-values; FDR adjustment via empirical Bayes is only planned future work
- Default p-values come from the Wilcoxon rank sum test; substituting DESeq is supported, but simulations showed the Wilcoxon-based version achieves higher accuracy than the DESeq-based version

## Evidence

- [abstract] U-curve created by plotting significant features vs mixing proportion: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing."
- [readme] plot_bin_psig invocation from README: "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [readme] expected decreasing trend rationale: "We compare the signal strength of top hits from the original data with their performance in permutations, and will observe an apparent decreasing trend"
- [abstract] DESeq extension for p-value source: "Our proposed method can also utilize p-values obtained from other testing methods, such as DESeq."
- [discussion] descriptive-only caveat: "Our paper is mainly designed for the general exploration and visualization of microbiome data, and does not come with a formal inference method."
