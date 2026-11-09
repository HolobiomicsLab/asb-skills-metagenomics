---
name: microbiome-group-difference-analysis
description: Use when a microbiome OTU/feature count table has a binary grouping factor
  and you need to characterize overall association strength and the robustness of
  significant features across progressive label permutations.
when_to_use_negative:
- 'The goal is formal inference: the method is ''mainly designed for the general exploration
  and visualization of microbiome data, and does not come with a formal inference
  method'' and does not control the false discovery rate or calculate adjusted p-values.'
- The grouping factor is not a binary variable — bin_permute_all is documented for
  a binary outcome ('"variable" denotes the name of binary outcome').
- 'top_pm is not smaller than the number of microbial features (README: ''"top_pm"
  must be less than the number of microbial features'').'
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
tools:
- name: ProgPermute
  role: R package implementing progressive permutation of the grouping factor and
    per-scenario differential testing via bin_permute_all
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Runtime environment for the ProgPermute package and its plotting functions
    (plot_bin_psig, plot_bin_pv, plot_bin_permute_sigcurve)
- name: R-shiny
  role: User-friendly Shiny interface exposing the progressive permutation workflow
    with visualizations (https://biostatistics.mdanderson.org/shinyapps/ProgPerm)
  repo: https://github.com/LyonsZhang/ProgPermute
- name: DESeq
  role: 'Alternative p-value source: the progressive permutation method ''can also
    utilize p-values obtained from other testing methods, such as DESeq'''
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:d7bd04826bb4acb1d13e847238580d73b22d42dce987b9dbbf4f5eb741b3aa29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# microbiome-group-difference-analysis

## Summary

Applies progressive permutation of a binary grouping factor in the ProgPermute R package: sample labels are progressively mixed between groups, differential abundance tests (default Wilcoxon rank sum, alpha=0.05) are rerun at each scenario, and the number of significant features is plotted against the proportion of mixing to yield a U-curve that conveys the strength of the overall microbiome–grouping association.

## When to use

Use when a microbiome OTU/feature count table has a binary grouping factor and you need to characterize overall association strength and the robustness of significant features across progressive label permutations.

## When NOT to use

- The goal is formal inference: the method is 'mainly designed for the general exploration and visualization of microbiome data, and does not come with a formal inference method' and does not control the false discovery rate or calculate adjusted p-values.
- The grouping factor is not a binary variable — bin_permute_all is documented for a binary outcome ('"variable" denotes the name of binary outcome').
- top_pm is not smaller than the number of microbial features (README: '"top_pm" must be less than the number of microbial features').

## Inputs

- Microbiome OTU/feature count data frame with a binary grouping column (e.g. testdata1, loaded via data(testdata1))
- Name of the binary outcome variable (e.g. variable="variable")
- Parameters: top_pm (must be less than the number of microbial features, e.g. 267), zoomn (e.g. 15), alpha (e.g. 0.05)

## Outputs

- Per-scenario significant-feature counts nsig(k) across progressive permutation mixing levels
- U-curve plot of number of significant features versus proportion of mixing (e.g. sigfeatures.eps via plot_bin_psig)
- Results object (alloutputs) for downstream plotting of p-value traces, scaled U-curve, fragility index, and progressive coverage

## How to apply

Load the ProgPermute package (install_github("LyonsZhang/ProgPermute")) and the dataset, e.g. data(testdata1). Run bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), where "variable" names the binary outcome and top_pm must be less than the number of microbial features; the function progressively exchanges sample labels and performs per-scenario differential tests with the default Wilcoxon rank sum test at alpha=0.05. Summarize per-scenario median p-values into the count of significant taxa nsig(k) at each progressive permutation scenario. Plot the number of significant features versus proportion of mixing with plot_bin_psig(alloutputs=results, psigyrange=c(0,170), savepsigfile="sigfeatures.eps"). A decreasing-then-U-shaped trend from the observed data toward the fully mixed scenario indicates true positives; the U-curve's depth and shape convey the strength of the overall association.

## Related tools

- **ProgPermute** (R package implementing progressive permutation of the grouping factor and per-scenario differential testing via bin_permute_all) — https://github.com/LyonsZhang/ProgPermute
- **R** (Runtime environment for the ProgPermute package and its plotting functions (plot_bin_psig, plot_bin_pv, plot_bin_permute_sigcurve))
- **R-shiny** (User-friendly Shiny interface exposing the progressive permutation workflow with visualizations (https://biostatistics.mdanderson.org/shinyapps/ProgPerm)) — https://github.com/LyonsZhang/ProgPermute
- **DESeq** (Alternative p-value source: the progressive permutation method 'can also utilize p-values obtained from other testing methods, such as DESeq')

## Examples

```
results <- bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05); plot_bin_psig(alloutputs=results, psigyrange=c(0,170), savepsigfile="sigfeatures.eps")
```

## Evaluation signals

- sigfeatures.eps exists as a valid EPS plot and shows a decreasing-then-U-shaped trend of significant-feature counts from the observed data toward the full permutation scenario, comparable to the deposited UCurve.png reference.
- Each per-scenario nsig(k) is consistent with the count of features with median p-value below alpha=0.05 in that permutation scenario.
- top_pm parameter satisfies the README constraint of being less than the total number of microbial features in the input table.
- Wilcoxon rank sum p-values are used by default; simulations show the Wilcoxon-based version 'achieves higher accuracy than the version based on DESeq'.
- A clearly decreasing initial trend in the U-curve indicates top hits from the original data behave as true positives under permutation.

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR control is only planned future work.
- AOI and AUMC measures are descriptive, not inferential — no formal inference method is provided.
- Ordering of features by fragility index is not perfect, though simulations show high accuracy in identifying differential features.
- Comparative findings depend on the alternative test: LEfSe was 'too generous, with a high number of false positives' in simulations.

## Evidence

- [abstract] U-curve from significant-feature counts: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing."
- [abstract] default Wilcoxon test: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [discussion] no FDR control: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
- [readme] core workflow call: "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [readme] top_pm constraint and binary variable: ""top_pm" must be less than the number of microbial features"
