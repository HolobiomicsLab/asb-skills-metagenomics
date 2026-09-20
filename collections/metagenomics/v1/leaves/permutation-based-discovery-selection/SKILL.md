---
name: permutation-based-discovery-selection
description: Use when a microbiome abundance table with a binary grouping variable
  is available and you must decide which top differential features are true discoveries
  rather than noise from label mixing, via full-permutation confidence intervals.
when_to_use_negative:
- You need confirmatory inference with false discovery rate control or adjusted p-values
  — the authors state the method does not control FDR and provides no formal inference.
- The grouping factor is not binary — the bin_* functions and coverage/interval logic
  are built for a binary outcome ('variable' denotes the name of binary outcome).
- You intend to set top_pm greater than or equal to the number of microbial features
  — the README requires top_pm to be less than the feature count.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ProgPermute
  role: R package providing bin_permute_best, plot_bin_permute_best, plot_bin_effectsize
    and dotplot_bin_sig used to run the full permutation scenario and select discoveries
  repo: https://github.com/LyonsZhang/ProgPerm
- name: R-shiny
  role: User-friendly interface that finalizes the results by listing the discoveries
    together with their effect sizes and individual abundances
  repo: https://github.com/LyonsZhang/ProgPerm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:9da307230b12bc2134fe5654fca7d67408368894dc22def2b8e08cabe8bc6447
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPerm
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# permutation-based-discovery-selection

## Summary

Selects robust microbiome feature discoveries by comparing p-values of the observed (unpermuted) data against p-values from the fully mixed (fully permuted) data, using the ProgPermute package, and assembles the final discoveries list with signed effect sizes and individual abundance plots.

## When to use

Use when a microbiome abundance table with a binary grouping variable is available and you must decide which top differential features are true discoveries rather than noise from label mixing, via full-permutation confidence intervals.

## When NOT to use

- You need confirmatory inference with false discovery rate control or adjusted p-values — the authors state the method does not control FDR and provides no formal inference.
- The grouping factor is not binary — the bin_* functions and coverage/interval logic are built for a binary outcome ('variable' denotes the name of binary outcome).
- You intend to set top_pm greater than or equal to the number of microbial features — the README requires top_pm to be less than the feature count.

## Inputs

- microbiome abundance table with a binary grouping factor column (e.g. testdata1 loaded via data(testdata1))
- name of the binary outcome variable (e.g. 'variable')
- top_pm (number of top features, must be less than the number of microbial features, e.g. 50)
- zoomn (e.g. 100) and alpha (e.g. 0.05) parameters

## Outputs

- pvalue_Coverageplot.eps and effectsize_Coverageplot.eps (95% quantile interval coverage plots from plot_bin_permute_best)
- signedeffectsize_plot.eps (signed effect sizes of discoveries from plot_bin_effectsize)
- individual abundance dot plots per discovery written to a folder (e.g. individual1)
- discoveries table (e.g. discoveries_table.csv) listing recommended features, effect sizes, and p-value evidence

## How to apply

Load the microbiome dataset (e.g. data(testdata1) shipped with ProgPermute) and run the full-permutation step with bin_permute_best(variable='<binary outcome name>', testdata=<data>, top_pm=50, zoomn=100, alpha=0.05), where top_pm must be less than the number of microbial features; features are tested by default with the Wilcoxon rank sum test. Identify discoveries as features whose -log10 p-values in the unpermuted scenario fall outside the 95% quantile intervals of the median -log10 p-values in the full permutation (Kf) scenario, visualized with plot_bin_permute_best (saving pvalue_Coverageplot.eps and effectsize_Coverageplot.eps). Compute signed effect sizes of the discoveries with plot_bin_effectsize (saving signedeffectsize_plot.eps), then produce per-feature abundance dot plots by lapply over best$goodpvname with dotplot_bin_sig into a folder such as 'individual1'. Finally assemble the recommended feature list, effect sizes, and p-value evidence into a discoveries table; the RShiny tool can finalize the same outputs.

## Related tools

- **ProgPermute** (R package providing bin_permute_best, plot_bin_permute_best, plot_bin_effectsize and dotplot_bin_sig used to run the full permutation scenario and select discoveries) — https://github.com/LyonsZhang/ProgPerm
- **R-shiny** (User-friendly interface that finalizes the results by listing the discoveries together with their effect sizes and individual abundances) — https://github.com/LyonsZhang/ProgPerm

## Examples

```
best<-bin_permute_best(variable="variable",testdata=testdata1,top_pm=50,zoomn=100,alpha=0.05)
```

## Evaluation signals

- Every feature listed as a discovery in the final table has an observed -log10 p-value outside the 95% quantile confidence interval of the full permutation scenario.
- The discovery list matches best$goodpvname returned by bin_permute_best, and the same names appear in the dot-plot folder (e.g. individual1).
- pvalue_Coverageplot.eps, effectsize_Coverageplot.eps and signedeffectsize_plot.eps are produced by plot_bin_permute_best and plot_bin_effectsize respectively.
- Signed effect sizes have consistent signs with the direction of the binary grouping for each reported discovery.
- top_pm used in the call is strictly less than the number of microbial features in the input table.

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values at this time.
- Measures such as AOI or AUMC are descriptive, not inferential, so discoveries require further statistical confirmation.
- Ordering of signals by fragility index is not perfect, although simulations show high accuracy in identifying differential features.

## Evidence

- [abstract] we recommend the identified features by comparing p-values in the observed data with p-values in the fully mixed data.: "we recommend the identified features by comparing p-values in the observed data with p-values in the fully mixed data."
- [readme] Real hits lie outside of 95% quantile intervals of full permuted hits: "Real hits lie outside of 95% quantile intervals of full permuted hits"
- [abstract] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [discussion] At this time, our method does not control the false discovery rate or calculate adjusted p-values.: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
