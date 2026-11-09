---
name: combinatorial-scenario-enumeration
description: Use when a two-group microbiome abundance table must be tested for association
  robustness across progressively mixed label permutations rather than a single global
  permutation test.
when_to_use_negative:
- The grouping factor has more than two levels — the progressive label-exchange scheme
  (bin_exchange, K=min(n1,n2)) is defined for binary outcomes only
- You need formal inference such as FDR-controlled adjusted p-values — the paper states
  the method 'does not control the false discovery rate or calculate adjusted p-values'
- Input is not a raw feature-by-sample count table with a binary outcome (e.g. already-permuted
  or summarized results) — scenario enumeration requires the original group labels
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ProgPermute
  role: R package implementing bin_permute_all, bin_exchange, and bin_multitest for
    progressive scenario enumeration and per-scenario differential testing
  repo: https://github.com/LyonsZhang/ProgPermute
- name: R
  role: runtime environment hosting the ProgPermute package and the dataset testdata1
- name: R-shiny
  role: interactive web front end for the progressive permutation analysis with visualizations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:b458cafaa72d1fab0a201093f042f650504994db53aa15713ad6d2dd9201d327
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LyonsZhang/ProgPermute
  tools:
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# combinatorial-scenario-enumeration

## Summary

Enumerates progressive label-mixing permutation scenarios for a two-group microbiome dataset, computing scenario bounds K=min(n1,n2) and Kf=ceil((n1*n2-1)/(n1+n2+2)) via the Vandermonde identity, and drawing label exchanges per scenario to enable differential testing under controlled mixing.

## When to use

Use when a two-group microbiome abundance table must be tested for association robustness across progressively mixed label permutations rather than a single global permutation test.

## When NOT to use

- The grouping factor has more than two levels — the progressive label-exchange scheme (bin_exchange, K=min(n1,n2)) is defined for binary outcomes only
- You need formal inference such as FDR-controlled adjusted p-values — the paper states the method 'does not control the false discovery rate or calculate adjusted p-values'
- Input is not a raw feature-by-sample count table with a binary outcome (e.g. already-permuted or summarized results) — scenario enumeration requires the original group labels

## Inputs

- microbiome abundance table with 267 microbial feature columns (testdata1 from ProgPermute R package)
- binary grouping variable name (e.g. 'variable')
- significance threshold psigalpha (default 0.05)

## Outputs

- one p-value matrix per permutation scenario k (p rows, one per feature), collectable as permutation_pvalues.rds
- per-scenario median p-values p_j^m(k) and 2.5-97.5% quantile intervals
- scenario range k=0..Kf with K=min(n1,n2) and Kf=ceil((n1*n2-1)/(n1+n2+2))

## How to apply

Load the packaged dataset (e.g. data(testdata1) from the ProgPermute R package) and extract the microbial feature columns and the binary grouping variable with group sizes n1 and n2. Compute the maximal permutation scenario K=min(n1,n2) and the full permutation scenario Kf=ceil((n1*n2-1)/(n1+n2+2)), derived from the Vandermonde identity sum_k C(n1,k)C(n2,k)=C(N,K). For each scenario k=0..Kf, construct permutations by randomly drawing k samples from each group and exchanging their labels (bin_exchange), performing nu=N*(log C(n1,k)+log C(n2,k)) rounded draws per scenario for k>=1 starting from a random seed. In each draw, run per-feature differential testing (Wilcoxon rank sum test by default, psigalpha=0.05) via bin_multitest, then summarize scenario p-value samples by medians and 2.5-97.5% quantile intervals, storing one p-value matrix per scenario. The rationale is that progressive mixing recovers missing label arrangements that a single full permutation cannot, producing a U-curve of nsig(k) versus mixing proportion that is lowest at the full permutation scenario Kf.

## Related tools

- **ProgPermute** (R package implementing bin_permute_all, bin_exchange, and bin_multitest for progressive scenario enumeration and per-scenario differential testing) — https://github.com/LyonsZhang/ProgPermute
- **R** (runtime environment hosting the ProgPermute package and the dataset testdata1)
- **R-shiny** (interactive web front end for the progressive permutation analysis with visualizations)

## Examples

```
results <- bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05)
```

## Evaluation signals

- Each scenario p-value matrix contains p rows (one per microbial feature, 267 for testdata1) with defined median p-values for every scenario k
- The number of significant features nsig(k) is lowest at the full permutation scenario Kf, where p-values approach 1
- Number of draws per scenario k>=1 equals rounded nu=N*(log C(n1,k)+log C(n2,k))
- Scenario bounds satisfy K=min(n1,n2) and Kf=ceil((n1*n2-1)/(n1+n2+2)) for the observed group sizes
- script_runs succeeds and outputs can be consumed by plot_bin_psig / plot_bin_pv without errors

## Limitations

- The method provides descriptive measures only (AOI, AUMC); no formal inference method is provided
- Does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR control is only planned future work
- Ordering of signals by fragility index is not perfect, although simulations show high identification accuracy
- top_pm must be less than the number of microbial features (e.g. 267 for testdata1)

## Evidence

- [abstract] we progressively permute the grouping factor labels of the microbiome samples and perform multiple differential abundance tests in each scenario: "we progressively permute the grouping factor labels of the microbiome samples and perform multiple differential abundance tests in each scenario"
- [methods] By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test.: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [readme] results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05): "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
- [discussion] At this time, our method does not control the false discovery rate or calculate adjusted p-values.: "At this time, our method does not control the false discovery rate or calculate adjusted p-values."
