---
name: microbiome-effect-size-visualization
description: Use when progressive permutation has flagged robust microbiome features
  (observed -log10 p-values outside the 95% quantile intervals of the fully permuted
  scenario) and you need signed effect sizes and individual abundance plots.
when_to_use_negative:
- Discovery set is empty because no features fell outside the 95% quantile intervals
  of the full permutation scenario — no effect size visualization is warranted
- The goal is to rank or filter features (use bin_fragility / bin_progresscoverage
  instead) rather than to characterize effect sizes of already-selected discoveries
- Data is not from a binary-grouping microbiome differential abundance analysis; the
  bin_* ProgPermute functions assume a binary outcome variable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: ProgPermute
  role: R package providing plot_bin_effectsize and dotplot_bin_sig to render signed
    effect size plots and individual abundance dot plots of discoveries
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Execution environment; effect size plots saved as EPS via R plotting functions
    from the ProgPermute package
- name: RShiny
  role: User-friendly web interface that finalizes the results by listing the discoveries
    together with their effect sizes and individual abundances
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:88a902973e5d01d0dbf1423479ae25c569313d5ee8f3d3c42c2d94d8905b5767
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  - RShiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# microbiome-effect-size-visualization

## Summary

Visualize the signed effect sizes and individual abundance patterns of microbiome discoveries identified by ProgPermute's progressive permutation, producing a signed effect size plot and per-feature abundance dot plots that finalize the discoveries table.

## When to use

Use when progressive permutation has flagged robust microbiome features (observed -log10 p-values outside the 95% quantile intervals of the fully permuted scenario) and you need signed effect sizes and individual abundance plots.

## When NOT to use

- Discovery set is empty because no features fell outside the 95% quantile intervals of the full permutation scenario — no effect size visualization is warranted
- The goal is to rank or filter features (use bin_fragility / bin_progresscoverage instead) rather than to characterize effect sizes of already-selected discoveries
- Data is not from a binary-grouping microbiome differential abundance analysis; the bin_* ProgPermute functions assume a binary outcome variable

## Inputs

- bestoutputs object returned by bin_permute_best (contains goodpvname, the list of discovered features)
- Microbiome abundance dataset loaded via data(testdata1) with a binary grouping variable named 'variable'
- Output folder name for per-feature dot plots (e.g. "individual1")

## Outputs

- signedeffectsize_plot.eps — plot of signed effect sizes of the discoveries
- Per-discovery abundance dot plots written into folder "individual1"
- Evidence (effect sizes, abundances, p-values) assembled into discoveries_table.csv

## How to apply

After running bin_permute_best(variable="variable", testdata=testdata1, top_pm=50, zoomn=100, alpha=0.05) to obtain the best features, call plot_bin_effectsize(bestoutputs=best, variable="variable", testdata=testdata1, saveesfile="signedeffectsize_plot.eps", espicdim=c(15,10)) to list the signed effect sizes of the discoveries. Then use lapply(best$goodpvname, dotplot_bin_sig, variable="variable", testdata=testdata1, folder="individual1") to produce individual abundance dot plots for each discovery in the folder 'individual1'. The signed effect sizes convey the direction and magnitude of each feature's association with the binary grouping variable, while the dot plots reveal per-sample abundance patterns. Together these outputs feed the recommended feature list assembled into discoveries_table.csv alongside p-value evidence.

## Related tools

- **ProgPermute** (R package providing plot_bin_effectsize and dotplot_bin_sig to render signed effect size plots and individual abundance dot plots of discoveries) — https://github.com/LyonsZhang/ProgPermute
- **R** (Execution environment; effect size plots saved as EPS via R plotting functions from the ProgPermute package)
- **RShiny** (User-friendly web interface that finalizes the results by listing the discoveries together with their effect sizes and individual abundances)

## Examples

```
best<-bin_permute_best(variable="variable",testdata=testdata1,top_pm=50,zoomn=100,alpha=0.05); plot_bin_effectsize(bestoutputs=best,variable="variable",testdata=testdata1,saveesfile="signedeffectsize_plot.eps",espicdim=c(15,10)); lapply(best$goodpvname,dotplot_bin_sig,variable="variable",testdata=testdata1,folder="individual1")
```

## Evaluation signals

- Every feature plotted by plot_bin_effectsize appears in best$goodpvname, i.e. its observed -log10 p-value lies outside the 95% quantile interval of the full permutation scenario
- signedeffectsize_plot.eps is produced with the requested dimensions (espicdim=c(15,10)) and shows a signed (directional) effect size for each discovery
- Folder "individual1" contains one abundance dot plot per discovered feature, matching the length of best$goodpvname
- Each discovery in discoveries_table.csv is accompanied by both its effect size and p-value evidence

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values, so effect size visualizations carry no FDR guarantee
- AOI and AUMC measures are meant to be descriptive; the paper is designed for general exploration and visualization and does not come with a formal inference method
- top_pm must be less than the number of microbial features, and discoveries depend on the alpha=0.05 threshold used in the upstream bin_permute_best call

## Evidence

- [readme] The method can evaluate the overall association between microbiome and the grouping factor, rank the robustness of the discovered microbes, and list the discoveries, their effect sizes, and individual abundances.: "The method can evaluate the overall association between microbiome and the grouping factor, rank the robustness of the discovered microbes, and list the discoveries, their effect sizes, and"
- [other] the RShiny tool finalizes the results by listing the discoveries together with their effect sizes and individual abundances: [evidence span withheld — anchor status: not_found]
- [readme] Real hits lie outside of 95% quantile intervals of full permuted hits: "Real hits lie outside of 95% quantile intervals of full permuted hits"
- [methods] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
