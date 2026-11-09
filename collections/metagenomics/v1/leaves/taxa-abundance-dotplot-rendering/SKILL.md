---
name: taxa-abundance-dotplot-rendering
description: Use when bin_permute_ has already produced a discoveries list ($goodpvname)
  and you need per-feature abundance dot plots of each recommended microbe relative
  to the binary grouping factor.
when_to_use_negative:
- Before the full-permutation step has been run — dotplot_bin_sig requires best$goodpvname;
  use bin_permute_all/plot_bin_psig first to assess the overall U-curve association.
- When the goal is formal FDR-controlled inference — the authors state the method
  'does not control the false discovery rate or calculate adjusted p-values' and the
  measures are descriptive.
- When the grouping outcome is not binary — the bin_* and dotplot_bin_sig functions
  take a binary outcome ('"variable" denotes the name of binary outcome').
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ProgPermute
  role: R package providing bin_permute_best (discovers features via full permutation)
    and dotplot_bin_sig (renders the per-feature abundance dot plots); installed with
    install_github("LyonsZhang/ProgPermute")
  repo: github.com/LyonsZhang/ProgPermute
- name: R
  role: Execution environment for the lapply call over best$goodpvname and for loading
    testdata1
- name: R-shiny
  role: User-facing RShiny tool wrapping the same workflow (discoveries, effect sizes,
    individual abundances) with visualizations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:c3728243594969c6d364a98b4c78b7879f343c465fd6fd76825d617b8b266a79
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# taxa-abundance-dotplot-rendering

## Summary

Renders individual abundance dot plots for each microbiome feature discovered by ProgPermute's full-permutation step, using dotplot_bin_sig over best$goodpvname to show per-sample abundance of each discovery stratified by the binary grouping variable. This is the final visualization step that accompanies the discoveries, their effect sizes, and p-value evidence.

## When to use

Use when bin_permute_best has already produced a discoveries list (best$goodpvname) and you need per-feature abundance dot plots of each recommended microbe relative to the binary grouping factor.

## When NOT to use

- Before the full-permutation step has been run — dotplot_bin_sig requires best$goodpvname; use bin_permute_all/plot_bin_psig first to assess the overall U-curve association.
- When the goal is formal FDR-controlled inference — the authors state the method 'does not control the false discovery rate or calculate adjusted p-values' and the measures are descriptive.
- When the grouping outcome is not binary — the bin_* and dotplot_bin_sig functions take a binary outcome ('"variable" denotes the name of binary outcome').

## Inputs

- testdata1 microbiome dataset loaded via data(testdata1) (feature-by-sample table with a binary outcome column named 'variable')
- Output object of bin_permute_best (list containing best$goodpvname, the names of the discovered features)
- Output folder name, e.g. "individual1", in the working directory set with setwd()

## Outputs

- One individual abundance dot plot per discovered feature, saved into the folder "individual1" (EPS/graphics files)
- Supporting coverage plots pvalue_Coverageplot.eps, effectsize_Coverageplot.eps, and signedeffectsize_plot.eps from the preceding steps
- Discoveries table (discoveries_table.csv) assembling recommended features, effect sizes, and p-value evidence

## How to apply

First run the full-permutation discovery step: best <- bin_permute_best(variable="variable", testdata=testdata1, top_pm=50, zoomn=100, alpha=0.05), which performs progressive permutation down to the full permutation scenario Kf and tests each feature with the default Wilcoxon rank sum test. Identify discoveries as features whose -log10 p-values in the unpermuted scenario fall outside the 95% quantile intervals of the full-permutation scenario (visualized via plot_bin_permute_best into pvalue_Coverageplot.eps and effectsize_Coverageplot.eps). Then map dotplot_bin_sig over the discoveries with lapply(best$goodpvname, dotplot_bin_sig, variable="variable", testdata=testdata1, folder="individual1"), writing one abundance dot plot per discovery into the folder "individual1". The rationale is that the final recommendations come from comparing observed-data p-values (signal) against fully mixed data p-values (noise), and the dot plots let a reviewer inspect each discovery's individual abundance pattern before assembling the discoveries_table.csv.

## Related tools

- **ProgPermute** (R package providing bin_permute_best (discovers features via full permutation) and dotplot_bin_sig (renders the per-feature abundance dot plots); installed with install_github("LyonsZhang/ProgPermute")) — github.com/LyonsZhang/ProgPermute
- **R** (Execution environment for the lapply call over best$goodpvname and for loading testdata1)
- **R-shiny** (User-facing RShiny tool wrapping the same workflow (discoveries, effect sizes, individual abundances) with visualizations)

## Examples

```
lapply(best$goodpvname,dotplot_bin_sig,variable="variable",testdata=testdata1,folder="individual1")
```

## Evaluation signals

- The output folder "individual1" contains exactly one abundance dot plot per feature listed in best$goodpvname, and no plots for features outside that list.
- Every plotted feature is also flagged as a discovery in plot_bin_permute_best's pvalue_Coverageplot.eps, i.e. its observed -log10 p-value lies outside the 95% quantile interval of the median -log10 p-values in the full permutation scenario.
- Abundance patterns in each dot plot are consistent with the signed effect sizes shown in signedeffectsize_plot.eps from plot_bin_effectsize.
- The set of discoveries is unchanged when re-derived from the same bin_permute_best output (deterministic given best$goodpvname).

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR adjustment is only planned future work.
- AOI/AUMC-type measures and the visualizations are descriptive, not inferential — the paper 'does not come with a formal inference method'.
- top_pm must be less than the number of microbial features in the input table (e.g. top_pm=50 with testdata1).
- Ordering of discoveries by robustness is not perfect, though simulations show high accuracy in identifying differential features.

## Evidence

- [readme] lapply call showing the invocation: "lapply(best$goodpvname,dotplot_bin_sig,variable="variable",testdata=testdata1,folder="individual1")"
- [readme] README step description: ">Show the dot plot of each individual discovery"
- [abstract] Discovery criterion from the abstract: "we recommend the identified features by comparing p-values in the observed data with p-values in the fully mixed data."
- [methods] Default statistical test: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [readme] Real-hit criterion supporting validation: ">Real hits lie outside of 95% quantile intervals of full permuted hits"
