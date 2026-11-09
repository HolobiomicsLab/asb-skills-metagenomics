---
name: microbiome-feature-robustness-ranking
description: Use when binary-grouped microbiome count data has yielded significant
  differential features and you need to rank how robust each discovery is to progressive
  permutation of grouping labels.
when_to_use_negative:
- You need formal statistical inference, FDR control, or adjusted p-values — the paper
  states the method does not control the false discovery rate or calculate adjusted
  p-values
- The grouping factor is continuous or multi-class rather than binary — bin_permute_all
  and bin_fragility operate on a binary outcome variable
- Input has no initially significant features to assess — bin_fragility requires lowindx
  of initially significant features from bin_true_initial
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ProgPermute
  role: R package providing bin_permute_all, bin_true_initial, and bin_fragility for
    progressive permutation and fragility-index computation
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: Execution environment in which the ProgPermute functions are run
- name: R-shiny
  role: Web interface exposing the fragility-index workflow with visualizations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:3c82ee6b0735b0a6d20d9d048960730513d4732e76943b6793de7cdfd8a0354b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# microbiome-feature-robustness-ranking

## Summary

Computes a fragility index for each discovered microbiome feature via progressive permutation: the minimum number of label-exchange permutation steps needed to render a significant feature nonsignificant, then ranks discoveries by robustness using the ProgPermute R package.

## When to use

Use when binary-grouped microbiome count data has yielded significant differential features and you need to rank how robust each discovery is to progressive permutation of grouping labels.

## When NOT to use

- You need formal statistical inference, FDR control, or adjusted p-values — the paper states the method does not control the false discovery rate or calculate adjusted p-values
- The grouping factor is continuous or multi-class rather than binary — bin_permute_all and bin_fragility operate on a binary outcome variable
- Input has no initially significant features to assess — bin_fragility requires lowindx of initially significant features from bin_true_initial

## Inputs

- Microbiome OTU/count matrix with a binary grouping factor (e.g., testdata1 from ProgPermute)
- Grouping variable name (e.g., "variable") and alpha threshold (default 0.05)
- top_pm parameter (must be less than the number of microbial features, e.g., 267)

## Outputs

- Per-feature fragility index (FI_j): minimum permutation steps until significance is lost
- Ranked table of discovered features in decreasing fragility-index order (e.g., fragility_index_table.csv)
- Fragility p-value trace figure (locationPvfragility.eps)

## How to apply

Load the count data with data(testdata1) from the ProgPermute package and run progressive permutation via bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), which exchanges k grouping labels between the two groups per scenario and computes Wilcoxon rank sum p-values (the default test, chosen as a robust nonparametric test) for every feature in each scenario. Derive the initially significant feature indices with bin_true_initial(variable="variable", testdata=testdata1, top_pm=267) to supply as lowindx. Compute per-feature fragility indices with bin_fragility(alloutputs=results, lowindx=intres$n, top_pm=50, lgndcol=2, yrange=c(0,7), savepvfile="locationPvfragility.eps", pvpicdim=c(15,7)), where FI_j is the minimum permutation step k at which the feature's median p-value crosses alpha=0.05. Optionally scale each index by the full permutation scenario, sFI_j = FI_j/K_f with K_f = ceil((n1*n2-1)/(n1+n2+2)), and sort discovered features in decreasing fragility-index order, since larger indices denote more stable taxa.

## Related tools

- **ProgPermute** (R package providing bin_permute_all, bin_true_initial, and bin_fragility for progressive permutation and fragility-index computation) — https://github.com/LyonsZhang/ProgPermute
- **R** (Execution environment in which the ProgPermute functions are run)
- **R-shiny** (Web interface exposing the fragility-index workflow with visualizations)

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05); intres<-bin_true_initial(variable="variable",testdata=testdata1,top_pm=267); fragility<-bin_fragility(alloutputs=results,lowindx=intres$n,top_pm=50,lgndcol=2,yrange=c(0,7),savepvfile="locationPvfragility.eps",pvpicdim=c(15,7))
```

## Evaluation signals

- Every fragility index satisfies 0 <= FI_j < K_f, where K_f = ceil((n1*n2-1)/(n1+n2+2)) is the full permutation scenario count
- Ranked table is sorted from most to least robust (decreasing fragility index)
- Features with FI_j = 0 lose significance at the first permutation step; more stable taxa have larger indices
- Initially significant indices from bin_true_initial (intres$n) match the features analyzed by bin_fragility (lowindx)
- All reported p-value medians cross the alpha=0.05 threshold exactly once along the progressive permutation path

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; FDR adjustment via empirical Bayes is only planned future work
- FI, AOI, and AUMC measures are descriptive, not inferential
- Ordering of signals by fragility index is not perfect, though simulations show differential features are identified with a high accuracy rate

## Evidence

- [abstract] We also define a fragility index to measure the robustness of the discoveries.: "We also define a fragility index to measure the robustness of the discoveries."
- [readme] fragility<-bin_fragility(alloutputs=results,lowindx=intres$n,top_pm=50,lgndcol=2,yrange=c(0,7),pvtitle=NULL,savepvfile="locationPvfragility.eps",pvpicdim=c(15,7)): "fragility<-bin_fragility(alloutputs=results,lowindx=intres$n,top_pm=50,lgndcol=2,yrange=c(0,7),pvtitle=NULL,savepvfile="locationPvfragility.eps",pvpicdim=c(15,7))"
- [readme] intres<-bin_true_initial(variable="variable",testdata=testdata1,top_pm=267): "intres<-bin_true_initial(variable="variable",testdata=testdata1,top_pm=267)"
- [discussion] At this time, our method does not control the false discovery rate or calculate adjusted p-values.: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
