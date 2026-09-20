---
name: confidence-interval-coverage-comparison
description: Use when you have progressive-permutation outputs from binary-outcome
  microbiome data and need to decide which top hits are real discoveries by comparing
  observed p-values against the fully permuted data's 95% quantile intervals.
when_to_use_negative:
- When the grouping outcome is not binary — the bin_permute_best / bin_permute_all
  functions target a binary variable and a different code path would be needed
- When you need formal inference or FDR control — the paper states the method 'does
  not control the false discovery rate or calculate adjusted p-values' and AOI/AUMC
  measures are descriptive only
- When top_pm is not smaller than the number of microbial features — the README notes
  'top_pm must be less than the number of microbial features'
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ProgPermute
  role: R package providing bin_permute_best, plot_bin_permute_best, plot_bin_effectsize,
    and dotplot_bin_sig used to run the full permutation and compare observed vs.
    fully permuted p-value confidence regions
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R-shiny
  role: User-friendly web interface with visualizations that finalizes the discoveries
    list with effect sizes and individual abundances
- name: DESeq
  role: Alternative differential testing method whose p-values can feed the same coverage
    comparison instead of the default Wilcoxon rank sum test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:f0269d6ff70a6f975a0754b9618511bc112923337dc2bb4523503980b71e7f20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# confidence-interval-coverage-comparison

## Summary

Calls microbiome feature discoveries by checking whether each feature's -log10 p-value from the unpermuted observed data falls outside the 95% quantile confidence interval of median -log10 p-values from the fully permuted (fully mixed) scenario in the ProgPermute progressive permutation workflow.

## When to use

Use when you have progressive-permutation outputs from binary-outcome microbiome data and need to decide which top hits are real discoveries by comparing observed p-values against the fully permuted data's 95% quantile intervals.

## When NOT to use

- When the grouping outcome is not binary — the bin_permute_best / bin_permute_all functions target a binary variable and a different code path would be needed
- When you need formal inference or FDR control — the paper states the method 'does not control the false discovery rate or calculate adjusted p-values' and AOI/AUMC measures are descriptive only
- When top_pm is not smaller than the number of microbial features — the README notes 'top_pm must be less than the number of microbial features'

## Inputs

- Binary-outcome microbiome feature table loaded as data(testdata1) or equivalent with a binary grouping variable column named 'variable'
- Output object from bin_permute_all (progressive permutation across mixing scenarios)
- Output object from bin_permute_best (full-permutation scenario Kf results, including per-feature p-values under full mixing)

## Outputs

- pvalue_Coverageplot.eps — coverage plot of -log10 p-values: observed vs. 95% quantile intervals of full permuted hits
- effectsize_Coverageplot.eps — coverage plot of effect sizes vs. fully permuted confidence region
- signedeffectsize_plot.eps — signed effect sizes of the discoveries
- Individual abundance dot plots per discovery (dotplot_bin_sig into folder 'individual1')
- Discoveries list (best$goodpvname) suitable for assembling a discoveries table with effect sizes and p-value evidence

## How to apply

After running progressive permutation with bin_permute_all on the binary grouping variable (e.g. data(testdata1), top_pm=267, zoomn=15, alpha=0.05), execute the full-permutation scenario with bin_permute_best(variable="variable", testdata=testdata1, top_pm=50, zoomn=100, alpha=0.05), which by default uses the Wilcoxon rank sum test for each feature. Pass the best-outputs to plot_bin_permute_best(top_pm=50, ...) to draw coverage plots comparing each feature's -log10 p-value and effect size in the unpermuted scenario against the 95% quantile intervals of the full permutation scenario, saving pvalue_Coverageplot.eps and effectsize_Coverageplot.eps. A feature is classified as a discovery ('real hit') when its observed -log10 p-value lies outside the 95% quantile interval of the fully permuted hits; plot_bin_effectsize then lists the signed effect sizes of the discoveries (signedeffectsize_plot.eps) and lapply over best$goodpvname with dotplot_bin_sig produces individual abundance dot plots (e.g. into folder "individual1"). The rationale is that true signals should remain extreme under no label mixing but drop toward the noise distribution once group labels are fully permuted, so divergence from the fully mixed confidence region indicates a robust association.

## Related tools

- **ProgPermute** (R package providing bin_permute_best, plot_bin_permute_best, plot_bin_effectsize, and dotplot_bin_sig used to run the full permutation and compare observed vs. fully permuted p-value confidence regions) — https://github.com/LyonsZhang/ProgPermute
- **R-shiny** (User-friendly web interface with visualizations that finalizes the discoveries list with effect sizes and individual abundances)
- **DESeq** (Alternative differential testing method whose p-values can feed the same coverage comparison instead of the default Wilcoxon rank sum test)

## Examples

```
best<-bin_permute_best(variable="variable",testdata=testdata1,top_pm=50,zoomn=100,alpha=0.05); cinfresults<-plot_bin_permute_best(bestoutputs=best,top_pm=50,savepvfile="pvalue_Coverageplot.eps",saveesfile="effectsize_Coverageplot.eps")
```

## Evaluation signals

- Every feature listed as a discovery has an observed -log10 p-value outside the 95% quantile interval of the median -log10 p-values in the full permutation scenario
- The number of discoveries does not exceed top_pm (e.g. 50) used in bin_permute_best
- Coverage plots pvalue_Coverageplot.eps and effectsize_Coverageplot.eps render without error and clearly separate discoveries from the fully permuted region
- The signed effect sizes in signedeffectsize_plot.eps are consistent with the direction of abundance differences visible in the individual dot plots in folder 'individual1'

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR control is only planned future work
- AOI and AUMC measures are descriptive, not inferential, and no formal inference method is provided
- Ordering of signals by fragility index is not perfect, though simulations show high identification accuracy
- By default the Wilcoxon rank sum test is used; the Wilcoxon-based version achieves higher accuracy than the DESeq-based version on simulated data

## Evidence

- [readme] Real hits lie outside of 95% quantile intervals of full permuted hits: "Real hits lie outside of 95% quantile intervals of full permuted hits"
- [abstract] we recommend the identified features by comparing p-values in the observed data with p-values in the fully mixed data.: "we recommend the identified features by comparing p-values in the observed data with p-values in the fully mixed data."
- [abstract] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [methods] the version based on the Wilcoxon test achieves higher accuracy than the version based on DESeq: "the version based on the Wilcoxon test achieves higher accuracy than the version based on DESeq"
- [discussion] our method does not control the false discovery rate or calculate adjusted p-values: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
