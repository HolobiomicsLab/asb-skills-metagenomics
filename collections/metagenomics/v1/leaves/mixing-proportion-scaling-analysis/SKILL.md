---
name: mixing-proportion-scaling-analysis
description: Use when you have a microbiome feature table with a binary grouping outcome
  and need to quantify overall association strength and discovery robustness by tracking
  significant-feature counts across progressive permutation mixing proportions (U-curve).
when_to_use_negative:
- You need FDR-controlled or adjusted p-values / formal inferential guarantees — the
  paper states the method does not control the false discovery rate and that AOI/AUMC
  measures are descriptive only
- The grouping outcome is not binary (the bin_* workflow, e.g., bin_permute_all with
  a 'variable' binary outcome, is built for two-group comparisons)
- top_pm would be greater than or equal to the number of microbial features — the
  README requires top_pm to be less than the feature count
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: ProgPermute
  role: R package providing bin_permute_all, plot_bin_psig, plot_bin_pv and plot_bin_permute_sigcurve
    that implement progressive permutation, mixing-proportion scaling, and the U-curve
    plots
  repo: github.com/LyonsZhang/ProgPermute
- name: DESeq
  role: 'Alternative p-value engine: setting the method argument to DESeq makes every
    progressive permutation scenario compute p-values with the DESeq2 package in R
    instead of the Wilcoxon rank sum test'
- name: R-shiny
  role: User-friendly web interface exposing the progressive permutation workflow
    with visualizations
  repo: https://biostatistics.mdanderson.org/shinyapps/ProgPerm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:655aa3f6d5130507fb77a80154e58cef4c69c3ccd443f73d80aca41a072fd5ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mixing-proportion-scaling-analysis

## Summary

Progressive-permutation analysis that plots the number (and, scaled by sample size, the proportion) of significant differential features against the proportion of mixing of grouping labels, producing a U-curve whose depth conveys the strength of the microbiome–grouping-factor association. P-values default to the Wilcoxon rank sum test but can be swapped for DESeq2.

## When to use

Use when you have a microbiome feature table with a binary grouping outcome and need to quantify overall association strength and discovery robustness by tracking significant-feature counts across progressive permutation mixing proportions (U-curve).

## When NOT to use

- You need FDR-controlled or adjusted p-values / formal inferential guarantees — the paper states the method does not control the false discovery rate and that AOI/AUMC measures are descriptive only
- The grouping outcome is not binary (the bin_* workflow, e.g., bin_permute_all with a 'variable' binary outcome, is built for two-group comparisons)
- top_pm would be greater than or equal to the number of microbial features — the README requires top_pm to be less than the feature count

## Inputs

- Microbiome feature/count table with a binary grouping outcome column (e.g., testdata1: outcome 'variable', 267 microbial features)
- Name of the binary outcome variable (e.g., 'variable')
- top_pm parameter (< number of microbial features, e.g., 267)
- zoomn and alpha parameters (e.g., zoomn=15, alpha=0.05)
- Optional alternative p-value source: method argument set to DESeq (DESeq2 package in R) instead of default Wilcoxon rank sum

## Outputs

- bin_permute_all results object containing per-scenario differential testing outputs across mixing proportions
- U-curve plot of the number of significant features versus proportion of mixing (EPS, e.g., sigfeatures.eps)
- P-value traces of top features across scenarios (EPS, e.g., pvfeatures.eps)
- Scaled U-curve of the proportion of significant features (EPS, e.g., bin_sigcurve.eps)

## How to apply

Load the feature table and binary outcome (e.g., data(testdata1) with the outcome column 'variable' and 267 microbial features), then run bin_permute_all(variable='variable', testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), keeping top_pm strictly less than the number of features; p-values are computed by the Wilcoxon rank sum test by default, or by DESeq2 if the method argument is set accordingly. Render the U-curve of significant-feature counts with plot_bin_psig, setting psigyrange to span the observed-to-full-permutation counts (e.g., c(0,170)) and savepsigfile to an EPS filename such as 'sigfeatures.eps'. Trace the p-values of top hits with plot_bin_pv(top_pm=267, savepvfile='pvfeatures.eps') and produce the scaled curve via plot_bin_permute_sigcurve(samsize=267, savepsigfile='bin_sigcurve.eps'), which rescales counts into the proportion of significant features versus the proportion of mixing. Judge success by whether the U-curve decreases from the observed data (k=0) toward the full permutation scenario (k=K_f); a near-flat curve is expected only if no group signal exists. Optionally follow with bin_true_initial and bin_fragility to rank discoveries by fragility index.

## Related tools

- **ProgPermute** (R package providing bin_permute_all, plot_bin_psig, plot_bin_pv and plot_bin_permute_sigcurve that implement progressive permutation, mixing-proportion scaling, and the U-curve plots) — github.com/LyonsZhang/ProgPermute
- **DESeq** (Alternative p-value engine: setting the method argument to DESeq makes every progressive permutation scenario compute p-values with the DESeq2 package in R instead of the Wilcoxon rank sum test)
- **R-shiny** (User-friendly web interface exposing the progressive permutation workflow with visualizations) — https://biostatistics.mdanderson.org/shinyapps/ProgPerm

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); sigsum<-plot_bin_permute_sigcurve(alloutputs=results,samsize=267,savepsigfile="bin_sigcurve.eps")
```

## Evaluation signals

- The U-curve of significant-feature counts decreases from the observed data (k=0) toward the full permutation scenario (k=K_f)
- A near-flat U-curve is expected only if no group signal exists between the microbiome and the grouping factor
- top_pm is strictly less than the number of microbial features (e.g., 267 for testdata1)
- The three EPS output files (e.g., sigfeatures.eps, pvfeatures.eps, bin_sigcurve.eps) exist after calling the plot functions with savepsigfile/savepvfile
- On simulated data, the Wilcoxon-based version should achieve higher accuracy than the DESeq-based version, so a DESeq run should be treated as an extension check rather than the default sensitivity reference

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; empirical-Bayes FDR adjustment is only planned future work
- AOI and AUMC measures are descriptive, not inferential — no formal inference method is provided
- The Wilcoxon-based version achieves higher accuracy than the DESeq-based version on simulated data, so DESeq substitution trades accuracy for flexibility
- Ordering of signals by fragility index is not perfect, although simulations show a high accuracy rate for identifying differential features

## Evidence

- [abstract] U-curve of significant features versus mixing proportion conveys association strength: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing."
- [methods] Default p-value engine is the Wilcoxon rank sum test: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [methods] Core progressive permutation invocation on testdata1: "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [discussion] No FDR control or adjusted p-values: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
- [readme] Progressive permutation workflow overview from README: "The proposed method progressively permutes the grouping factor labels of microbiome and performs multiple differential abundance tests in each scenario."
