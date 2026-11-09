---
name: p-value-trace-visualization
description: Use when you have run progressive permutation of a binary grouping factor
  over a microbiome feature table and need to visually assess whether the most differential
  features from the original data retain signal strength (as -log10 p-values) as mixing
  approaches full permutation.
when_to_use_negative:
- You need formal inference, FDR control, or adjusted p-values — the authors state
  the method is for general exploration and visualization and does not control the
  false discovery rate or calculate adjusted p-values
- You want the overall-association U-curve of number of significant features versus
  proportion of mixing rather than per-feature signal traces (use plot_bin_psig /
  plot_bin_permute_sigcurve instead)
- The grouping factor is not binary, or the requested top_pm is not less than the
  total number of microbial features
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: ProgPermute
  role: R package providing bin_permute_all and plot_bin_pv for progressive permutation
    and the p-value trace plot
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Implementation language; Wilcoxon rank sum test is the default differential
    test per permutation scenario
- name: R-shiny
  role: User-friendly web interface with visualizations including the p-value trace
    plot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:838dee01f5080db577c2246247d6a1424dbee62226c034e0835b86848c1a81e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# p-value-trace-visualization

## Summary

Generates the ProgPermute p-value trace plot, which ranks features by their observed p-value in the unpermuted data and plots -log10 of the median p-value with 95% quantile intervals across progressive permutation scenarios (k=0 to Kf), so true-positive microbiome hits show a visually apparent decreasing signal trend.

## When to use

Use when you have run progressive permutation of a binary grouping factor over a microbiome feature table and need to visually assess whether the most differential features from the original data retain signal strength (as -log10 p-values) as mixing approaches full permutation.

## When NOT to use

- You need formal inference, FDR control, or adjusted p-values — the authors state the method is for general exploration and visualization and does not control the false discovery rate or calculate adjusted p-values
- You want the overall-association U-curve of number of significant features versus proportion of mixing rather than per-feature signal traces (use plot_bin_psig / plot_bin_permute_sigcurve instead)
- The grouping factor is not binary, or the requested top_pm is not less than the total number of microbial features

## Inputs

- alloutputs object returned by bin_permute_all (p-values per feature per permutation scenario k=0..Kf)
- Microbiome feature (OTU) count table with a binary grouping factor, e.g. data(testdata1) from the ProgPermute package
- Plot parameters: top_pm (number of top-ranked features, must be less than the number of microbial features), pvyrange (e.g. c(0,7)), pvpicdim (e.g. c(10,7))

## Outputs

- EPS figure pvfeatures.eps showing -log10 median p-value traces with 95% quantile intervals across permutation scenarios for the top_pm most differential features
- Returned plot object (pvloc) from plot_bin_pv

## How to apply

After loading the example microbiome dataset with data(testdata1) and running progressive permutation via bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), which performs differential testing (Wilcoxon rank sum by default) at each mixing scenario k from 0 to Kf, call plot_bin_pv with top_pm=267, pvyrange=c(0,7), savepvfile="pvfeatures.eps", and pvpicdim=c(10,7). The function ranks features by their observed p-value and plots the -log10 median p-value with 95% quantile intervals at each permutation scenario. The rationale is that top hits that are true positives in the original data should start high at k=0 (no permutation) and decrease toward nonsignificance as labels are progressively mixed; persistence of high signal under near-full permutation suggests the hit may not be robust. Use the pvyrange ceiling (e.g. 7, corresponding to -log10 p of the smallest resolvable p-value) to keep traces of highly significant features visible on one figure.

## Related tools

- **ProgPermute** (R package providing bin_permute_all and plot_bin_pv for progressive permutation and the p-value trace plot) — https://github.com/LyonsZhang/ProgPermute
- **R** (Implementation language; Wilcoxon rank sum test is the default differential test per permutation scenario)
- **R-shiny** (User-friendly web interface with visualizations including the p-value trace plot)

## Examples

```
pvloc<-plot_bin_pv(alloutputs=results,top_pm=267,pvtitle="",pvyrange=c(0,7),savepvfile="pvfeatures.eps",pvpicdim=c(10,7))
```

## Evaluation signals

- The output file pvfeatures.eps exists in the working directory
- Traces of the more significant variables start high at k=0 (no permutation) and decrease toward nonsignificance as mixing approaches the full permutation scenario, for true-positive features
- Traces are plotted as -log10 median p-values with 95% quantile intervals, and the y-axis respects pvyrange=c(0,7)
- Exactly top_pm=267 features (fewer than the total number of microbial features) appear, ranked by their observed p-value in the original data

## Limitations

- The method is designed for general exploration and visualization and does not come with a formal inference method; AOI/AUMC-type measures are descriptive, not inferential
- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR adjustment is only planned future work
- Ordering of signals by robustness is not perfect, although simulations show high accuracy in identifying differential features
- top_pm must be less than the number of microbial features in the input table

## Evidence

- [readme] show the traces of Pvalues: ">show the traces of Pvalues
```R
pvloc<-plot_bin_pv(alloutputs=results,top_pm=267,pvtitle="",pvyrange=c(0,7),savepvfile ="pvfeatures.eps""
- [readme] decreasing trend for true positives: "We compare the signal strength of top hits from the original data with their performance in permutations, and will observe an apparent decreasing trend if these top hits are true positives"
- [abstract] Wilcoxon default per scenario: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [discussion] no FDR control: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
- [readme] top_pm constraint: ">run progressive permutation. "variable" denotes the name of binary outcome. "top_pm" must be less than the number of microbial features"
