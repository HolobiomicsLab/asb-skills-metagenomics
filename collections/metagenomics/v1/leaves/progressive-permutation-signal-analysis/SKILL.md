---
name: progressive-permutation-signal-analysis
description: Use when you have a feature-by-sample microbiome count table with a binary
  grouping factor and need to assess whether top differential features are robust
  to label permutation, or to rank them by fragility index.
when_to_use_negative:
- You need formal inference or FDR-controlled adjusted p-values — the paper states
  the method does not control the false discovery rate and its measures are descriptive
  only
- The dataset has no binary grouping factor to progressively permute, or top_pm would
  exceed the number of microbial features
- You need an inferential effect-size test rather than exploratory visualization of
  discovery robustness (the paper positions this as exploration and visualization)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ProgPermute
  role: R package providing bin_permute_all, plot_bin_pv, bin_true_initial, and bin_fragility
    to run progressive permutation, trace -log10 p-values, and compute fragility indices
  repo: github.com/LyonsZhang/ProgPermute
- name: R-shiny
  role: User-friendly web application wrapping the progressive permutation workflow
    with visualizations
  repo: https://biostatistics.mdanderson.org/shinyapps/ProgPerm
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:769682697636016fd03d1b928114f098e1aec4828eca104241eab7ed8c7ae250
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# progressive-permutation-signal-analysis

## Summary

Progressive permutation of microbiome grouping labels (ProgPermute) to visualize the decay of -log10 p-value traces for top hits across mixing scenarios and to rank discoveries by a fragility index, measuring robustness of differential-abundance findings.

## When to use

Use when you have a feature-by-sample microbiome count table with a binary grouping factor and need to assess whether top differential features are robust to label permutation, or to rank them by fragility index.

## When NOT to use

- You need formal inference or FDR-controlled adjusted p-values — the paper states the method does not control the false discovery rate and its measures are descriptive only
- The dataset has no binary grouping factor to progressively permute, or top_pm would exceed the number of microbial features
- You need an inferential effect-size test rather than exploratory visualization of discovery robustness (the paper positions this as exploration and visualization)

## Inputs

- Microbiome feature-by-sample count table (e.g. OTU counts in testdata1 from the ProgPermute R package)
- Binary grouping factor column (e.g. "variable" in testdata1)
- Parameters: top_pm (number of top features, must be less than the number of microbial features), zoomn, alpha=0.05

## Outputs

- P-value trace plot (pvfeatures.eps) of -log10 median p-values with 95% quantile intervals per permutation scenario
- Per-feature fragility indices FI_j = min_k(p^m_j(k) > 0.05) from median p-values across permutation scenarios
- Scaled fragility indices sFI_j = FI_j/K_f with K_f = ceil((n1*n2-1)/(n1+n2+2))
- Ranked table of discovered features ordered from most to least robust (e.g. fragility_index_table.csv)

## How to apply

Load the example data with data(testdata1) in R, then run progressive permutation via bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), which exchanges grouping labels between the two groups at each scenario k from 0 to Kf and computes Wilcoxon rank sum p-values (the default test because it is a robust nonparametric test) for every feature per scenario; top_pm must be less than the number of microbial features. Generate the p-value trace plot with plot_bin_pv (e.g. pvyrange=c(0,7), savepvfile="pvfeatures.eps"), which ranks features by observed p-value and plots -log10 median p-values with 95% quantile intervals across permutation scenarios; a visually decreasing trend from k=0 toward full permutation indicates the top hits are true positives. For per-feature robustness, derive initially significant indices with bin_true_initial (top_pm=267) and compute fragility with bin_fragility, where FI_j = min_k(p^m_j(k) > 0.05) is taken from median p-values across scenarios. Scale each index by the full permutation scenario, sFI_j = FI_j/K_f with K_f = ceil((n1*n2-1)/(n1+n2+2)), and sort discoveries in decreasing fragility-index order, since larger indices denote more stable taxa.

## Related tools

- **ProgPermute** (R package providing bin_permute_all, plot_bin_pv, bin_true_initial, and bin_fragility to run progressive permutation, trace -log10 p-values, and compute fragility indices) — github.com/LyonsZhang/ProgPermute
- **R-shiny** (User-friendly web application wrapping the progressive permutation workflow with visualizations) — https://biostatistics.mdanderson.org/shinyapps/ProgPerm

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); pvloc<-plot_bin_pv(alloutputs=results,top_pm=267,pvyrange=c(0,7),savepvfile="pvfeatures.eps",pvpicdim=c(10,7))
```

## Evaluation signals

- The output file pvfeatures.eps exists after plot_bin_pv completes
- In the trace plot, p-value traces of the most significant variables start high at k=0 (no permutation) and decrease toward nonsignificance as mixing approaches the full permutation scenario
- Every computed fragility index satisfies 0 <= FI_j < K_f, and the ranked table is ordered from most to least robust (decreasing fragility index)
- The U-curve of number of significant features versus proportion of mixing is produced (e.g. via plot_bin_psig with psigyrange=c(0,170)), consistent with an overall association between microbiome and the grouping factor

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; FDR adjustment via empirical Bayes is only planned future work
- Measures such as AOI or AUMC are descriptive, not inferential; the paper provides no formal inference method
- Ordering of signals by fragility index is not perfect, although simulations show high accuracy in identifying differential features

## Evidence

- [readme] run progressive permutation: ">run progressive permutation. "variable" denotes the name of binary outcome. "top_pm" must be less than the number of microbial features"
- [readme] We compare the signal strength of top hits from the original data with their performance in permutations, and will observe an apparent decreasing trend if these top hits are true positives: "We compare the signal strength of top hits from the original data with their performance in permutations, and will observe an apparent decreasing trend if these top hits are true positives"
- [abstract] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [abstract] A fragility index is defined to measure the robustness of the discoveries.: "We also define a fragility index to measure the robustness of the discoveries."
- [abstract] the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing: "the proposed method creates a U-curve when plotting the number of significant features versus the proportion of mixing"
