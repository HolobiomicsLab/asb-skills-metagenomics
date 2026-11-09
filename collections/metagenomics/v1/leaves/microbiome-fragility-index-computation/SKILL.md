---
name: microbiome-fragility-index-computation
description: Use when a set of differentially abundant microbiome features from a
  binary-group comparison must be ranked by robustness, by counting how many progressive
  label-permutation steps flip each feature to nonsignificance.
when_to_use_negative:
- The method does not control the false discovery rate or calculate adjusted p-values
  — do not use fragility indices as a substitute for FDR-corrected significance testing
- Do not apply when the grouping factor is not a binary variable, since bin_permute_all/bin_true_initial/bin_fragility
  assume a binary outcome (e.g. 'variable')
- AOI/AUMC and fragility measures are descriptive, not inferential — do not cite them
  as formal statistical inference of association
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: ProgPermute
  role: R package providing bin_permute_all, bin_true_initial, and bin_fragility used
    to run progressive permutation and compute per-feature fragility indices
  repo: https://github.com/LyonsZhang/ProgPerm
- name: R
  role: Execution environment in which the ProgPermute example workflow (Execute_ProgPermute.R)
    is run
- name: R-shiny
  role: Interactive interface exposing the fragility index workflow with visualizations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:d3eb0ca6d64cc1b01020a04a68c5b54c4002c1c76b151125d1692dd7b65f43f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPerm
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# microbiome-fragility-index-computation

## Summary

Computes the ProgPermute fragility index — the minimum number of progressive permutation steps k at which a discovered microbiome feature's median Wilcoxon p-value crosses the 0.05 significance cutoff — to rank discovered taxa from most to least robust against label mixing.

## When to use

Use when a set of differentially abundant microbiome features from a binary-group comparison must be ranked by robustness, by counting how many progressive label-permutation steps flip each feature to nonsignificance.

## When NOT to use

- The method does not control the false discovery rate or calculate adjusted p-values — do not use fragility indices as a substitute for FDR-corrected significance testing
- Do not apply when the grouping factor is not a binary variable, since bin_permute_all/bin_true_initial/bin_fragility assume a binary outcome (e.g. 'variable')
- AOI/AUMC and fragility measures are descriptive, not inferential — do not cite them as formal statistical inference of association

## Inputs

- Microbiome OTU/feature count matrix with a binary grouping variable (e.g. ProgPermute testdata1)
- Progressive permutation results object from bin_permute_all
- Indices of initially significant features from bin_true_initial (lowindx = intres$n)
- Significance threshold alpha = 0.05 and top_pm cutoff (e.g. 267 for permutation, 50 for fragility display)

## Outputs

- Per-feature fragility index values FI_j (minimum permutation steps to lose significance)
- Scaled fragility indices sFI_j = FI_j/K_f for cross-dataset comparison
- Ranked list of discovered features ordered from most to least robust
- Median p-value trace plot per discovered feature (locationPvfragility.eps)

## How to apply

Load the count matrix (e.g. data(testdata1) from ProgPermute) with a binary grouping variable and run progressive permutation via bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), which exchanges k group labels between the two groups per scenario and computes Wilcoxon rank sum p-values for every feature. Identify initially significant features with bin_true_initial(variable="variable", testdata=testdata1, top_pm=267) to supply as lowindx. Call bin_fragility(alloutputs=results, lowindx=intres$n, top_pm=50, lgndcol=2, yrange=c(0,7), savepvfile="locationPvfragility.eps", pvpicdim=c(15,7)) so that FI_j = min_k such that the median p-value p_j(k) exceeds alpha=0.05. Optionally scale each index by the full-permutation scenario (sFI_j = FI_j/K_f, K_f = ceil((n1*n2-1)/(n1+n2+2))) and sort features in decreasing fragility-index order, since larger indices denote more stable taxa. Output the ranked table (e.g. fragility_index_table.csv) and the fragility p-value plot.

## Related tools

- **ProgPermute** (R package providing bin_permute_all, bin_true_initial, and bin_fragility used to run progressive permutation and compute per-feature fragility indices) — https://github.com/LyonsZhang/ProgPerm
- **R** (Execution environment in which the ProgPermute example workflow (Execute_ProgPermute.R) is run)
- **R-shiny** (Interactive interface exposing the fragility index workflow with visualizations)

## Examples

```
intres<-bin_true_initial(variable="variable",testdata=testdata1,top_pm=267); fragility<-bin_fragility(alloutputs=results,lowindx=intres$n,top_pm=50,lgndcol=2,yrange=c(0,7),savepvfile="locationPvfragility.eps",pvpicdim=c(15,7))
```

## Evaluation signals

- Every computed FI_j satisfies 0 <= FI_j < K_f, where K_f = ceil((n1*n2-1)/(n1+n2+2)) is the full-permutation scenario count
- The ranked table is sorted in strictly decreasing fragility-index order (most to least robust feature)
- Every feature in the fragility output appears in the initially significant set lowindx = intres$n from bin_true_initial
- Median p-value of each feature at k = 0 is significant (p < 0.05) and at k = FI_j is nonsignificant (p > 0.05) in the locationPvfragility.eps trace plot
- Scaled indices satisfy 0 <= sFI_j < 1

## Limitations

- Ordering of signals by fragility index is not perfect: 'Although the ordering results are not perfect, we show through simulations that the proposed method can identify the differential features with a high accuracy rate.'
- No formal inference method is provided; fragility index and related measures (AOI, AUMC) are descriptive only
- The method does not control the false discovery rate or calculate adjusted p-values; empirical-Bayes FDR adjustment is planned future work
- top_pm must be less than the number of microbial features (e.g. top_pm=267 for testdata1, top_pm=50 for the fragility display)

## Evidence

- [abstract] We also define a fragility index to measure the robustness of the discoveries.: "We also define a fragility index to measure the robustness of the discoveries."
- [methods] fragility<-bin_fragility(alloutputs=results,lowindx=intres$n,top_pm=50,lgndcol=2,yrange=c(0,7): "fragility<-bin_fragility(alloutputs=results,lowindx=intres$n,top_pm=50,lgndcol=2,yrange=c(0,7)"
- [readme] intres<-bin_true_initial(variable="variable",testdata=testdata1,top_pm=267): "intres<-bin_true_initial(variable="variable",testdata=testdata1,top_pm=267)"
- [readme] The method can evaluate the overall association between microbiome and the grouping factor, rank the robustness of the discovered microbes: "The method can evaluate the overall association between microbiome and the grouping factor, rank the robustness of the discovered microbes"
