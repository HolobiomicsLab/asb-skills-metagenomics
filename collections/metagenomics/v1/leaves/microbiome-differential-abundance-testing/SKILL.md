---
name: microbiome-differential-abundance-testing
description: Use when a binary or continuous grouping factor and a microbiome feature
  table are available and you need to test differential abundance while quantifying
  robustness of individual features and the overall microbiome-group association.
when_to_use_negative:
- The goal requires formal inference, FDR control, or adjusted p-values — the article
  states the method 'does not control the false discovery rate or calculate adjusted
  p-values' and AOI/AUMC are descriptive only
- The task is unsupervised ordination or community-structure comparison (NMDS/PCoA/PERMANOVA)
  rather than per-feature differential testing with robustness assessment
- top_pm exceeds the number of microbial features — the README notes top_pm must be
  less than the number of features
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ProgPermute
  role: R package implementing progressive permutation drivers (bin_permute_all, bin_permute_best,
    con_permute_all) and plotting functions (plot_bin_psig, plot_bin_pv, plot_bin_permute_sigcurve,
    plot_bin_permute_best, plot_bin_effectsize)
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R-shiny
  role: User-friendly web interface wrapping the workflow with visualizations
- name: DESeq
  role: 'Alternative p-value source: bin_permute_all can be run with method set to
    DESeq so p-values in every scenario are computed by DESeq2 instead of the default
    Wilcoxon rank sum test'
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  - task_006
  - task_007
  - task_001
  - task_008
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:1591d86967cefa1327571ec9e0e5848b8149c37f3f8ee469b61d940e852bfaad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# microbiome-differential-abundance-testing

## Summary

A progressive permutation workflow (ProgPerm/ProgPermute) for testing microbiome differential abundance that assesses the robustness of discoveries by permuting grouping factor labels across mixing scenarios and comparing observed p-values against fully permuted (noise) p-values.

## When to use

Use when a binary or continuous grouping factor and a microbiome feature table are available and you need to test differential abundance while quantifying robustness of individual features and the overall microbiome-group association.

## When NOT to use

- The goal requires formal inference, FDR control, or adjusted p-values — the article states the method 'does not control the false discovery rate or calculate adjusted p-values' and AOI/AUMC are descriptive only
- The task is unsupervised ordination or community-structure comparison (NMDS/PCoA/PERMANOVA) rather than per-feature differential testing with robustness assessment
- top_pm exceeds the number of microbial features — the README notes top_pm must be less than the number of features

## Inputs

- Microbiome feature/OTU count table with a binary grouping column (e.g. testdata1: 267 microbial features, outcome 'variable')
- Grouping factor(s): binary outcomes via bin_permute_* functions or continuous outcomes via con_permute_all
- Significance level alpha (default 0.05) and top_pm (< total number of features)

## Outputs

- Per-scenario median p-values with 2.5-97.5% quantile intervals for every feature
- U-curve of significant-feature counts (nsig(k)) versus proportion of mixing (sigfeatures.eps) and scaled U-curve (bin_sigcurve.eps)
- P-value trace plots of top hits (pvfeatures.eps)
- AUMC per grouping factor for ranking overall association strength
- Discoveries list with signed effect sizes and individual abundance dot plots (discoveries, signedeffectsize_plot.eps, individual1/ dot plots)
- Fragility index per discovered feature

## How to apply

Load the feature table and grouping variable (e.g. data(testdata1) with 267 features and outcome 'variable') in R and run bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), which progressively exchanges sample labels between groups from k=0 up to the full permutation scenario Kf = ceil((n1*n2-1)/(n1+n2+2)) and runs per-feature differential tests at each scenario (Wilcoxon rank sum by default at alpha=0.05; DESeq or, for continuous outcomes, Kendall's tau and Spearman correlation can be substituted). Summarize per-scenario median p-values and 2.5-97.5% quantile intervals, then plot the U-curve of significant-feature counts versus mixing proportion with plot_bin_psig and the p-value traces with plot_bin_pv. Quantify the overall association with the AUMC (area under the scaled U-curve of nsig(k)/p), which can rank multiple grouping factors (e.g. location AUMC 0.253 > gender AUMC 0.035). For final discoveries, run bin_permute_best to reach the full permutation scenario and call features as robust hits only when their observed -log10 p-values fall outside the 95% quantile intervals of the fully permuted hits; optionally compute a fragility index via bin_true_initial and bin_fragility.

## Related tools

- **ProgPermute** (R package implementing progressive permutation drivers (bin_permute_all, bin_permute_best, con_permute_all) and plotting functions (plot_bin_psig, plot_bin_pv, plot_bin_permute_sigcurve, plot_bin_permute_best, plot_bin_effectsize)) — https://github.com/LyonsZhang/ProgPermute
- **R-shiny** (User-friendly web interface wrapping the workflow with visualizations)
- **DESeq** (Alternative p-value source: bin_permute_all can be run with method set to DESeq so p-values in every scenario are computed by DESeq2 instead of the default Wilcoxon rank sum test)

## Examples

```
library(ProgPermute); data(testdata1); results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); sigloc<-plot_bin_psig(alloutputs=results,psigyrange=c(0,170),savepsigfile="sigfeatures.eps",psigpicdim=c(10,7))
```

## Evaluation signals

- sigfeatures.eps exists and shows a decreasing then U-shaped trend of significant-feature counts from observed data (k=0) toward the full permutation scenario Kf, comparable to the deposited UCurve.png
- In pvfeatures.eps, traces of more significant variables start high at k=0 and decrease toward nonsignificance as mixing approaches full permutation (expected for true positives)
- Every discovery listed as a hit has an observed -log10 p-value outside the 95% quantile interval of the full permutation scenario's median -log10 p-values
- AUMC values lie between 0 and 1, are near 0 when no true group difference exists, and reproduce known orderings (e.g. location AUMC 0.253 > gender AUMC 0.035)
- nsig(k) is lowest at the full permutation scenario Kf = ceil((n1*n2-1)/(n1+n2+2)), where p-values approach 1

## Limitations

- The method does not come with a formal inference method; measures are for general exploration and visualization
- AOI and AUMC are descriptive, not inferential; the method does not control false discovery rate or calculate adjusted p-values (empirical Bayes FDR control is planned future work)
- The version based on DESeq achieves lower accuracy on simulated data than the default Wilcoxon version
- Ordering of signals by fragility index is not perfect, though simulations show high identification accuracy
- LEfSe comparisons show some methods are too generous with high false positives — test choice matters

## Evidence

- [abstract] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [abstract] the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing.: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing."
- [methods] results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05): "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [readme] Real hits lie outside of 95% quantile intervals of full permuted hits: "Real hits lie outside of 95% quantile intervals of full permuted hits"
- [discussion] Our method does not control the false discovery rate or calculate adjusted p-values.: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
