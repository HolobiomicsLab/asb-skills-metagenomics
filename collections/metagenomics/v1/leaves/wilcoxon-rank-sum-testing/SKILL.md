---
name: wilcoxon-rank-sum-testing
description: Use when comparing microbial feature abundances between two binary groups
  from a 267-feature OTU/count table (e.g., testdata1) and you need robust nonparametric
  p-values per scenario of progressive permutation.
when_to_use_negative:
- When you need false discovery rate control or adjusted p-values — the method explicitly
  does not control the FDR or calculate adjusted p-values
- When the grouping variable is not binary or you require a formal inferential test
  on the permutation results — AOI/AUMC measures are descriptive only
- When you specifically want a parametric negative binomial model — DESeq is supported
  as an alternative p-value source, but the article reports the Wilcoxon version achieves
  higher accuracy on simulated data
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: ProgPermute
  role: R package providing bin_permute_all and bin_multitest that perform progressive
    label exchange and per-feature Wilcoxon rank sum testing in each scenario
  repo: github.com/LyonsZhang/ProgPermute
- name: R
  role: Implementation language; testdata1 is loaded in R and p-value matrices are
    saved as permutation_pvalues.rds
- name: R-shiny
  role: User-friendly interface exposing the progressive permutation and Wilcoxon-based
    testing workflow with visualizations
- name: DESeq
  role: Alternative p-value source that can replace the Wilcoxon default, though with
    lower accuracy on simulations
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04061-3
    title: 'ProgPerm: Progressive permutation for a dynamic representation of the
      robustness of microbiome discoveries'
  source_content_hash: sha256:e5a77db5d03095e0674bfe37bf6cbda72f075c06341b0593655b84baab27f38c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq
  - ProgPermute
  - R
  - R-shiny
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# wilcoxon-rank-sum-testing

## Summary

Applies the Wilcoxon rank sum test, the default differential abundance test in ProgPerm's progressive permutation, to compute per-feature p-values between two microbiome groups in each permutation scenario.

## When to use

Use when comparing microbial feature abundances between two binary groups from a 267-feature OTU/count table (e.g., testdata1) and you need robust nonparametric p-values per scenario of progressive permutation.

## When NOT to use

- When you need false discovery rate control or adjusted p-values — the method explicitly does not control the FDR or calculate adjusted p-values
- When the grouping variable is not binary or you require a formal inferential test on the permutation results — AOI/AUMC measures are descriptive only
- When you specifically want a parametric negative binomial model — DESeq is supported as an alternative p-value source, but the article reports the Wilcoxon version achieves higher accuracy on simulated data

## Inputs

- testdata1 (R dataset of 267 microbial feature count columns plus a binary grouping variable 'variable', loaded via data(testdata1))
- Binary grouping factor labels of microbiome samples
- alpha=0.05 significance threshold (psigalpha) for differential testing

## Outputs

- Per-scenario p-value matrices (one per permutation scenario k=0..Kf, p rows per feature, with median p-values and 2.5-97.5% quantile intervals)
- Permutation_pvalues.rds collection of per-scenario p-value matrices
- Inputs for U-curve (plot_bin_psig), p-value trace (plot_bin_pv), and fragility index (bin_fragility) outputs

## How to apply

Run ProgPermute's bin_permute_all(variable="variable", testdata=testdata1, top_pm=267, zoomn=15, alpha=0.05), which progressively exchanges k grouping labels between the two groups (k from 0 to Kf=ceil((n1*n2-1)/(n1+n2+2))) and, in each draw of every scenario, runs the Wilcoxon rank sum test per feature via bin_multitest with psigalpha=0.05 to obtain p_j(k) for all j=1..p. The Wilcoxon test is chosen by default because it is a robust nonparametric test, and simulation results showed the Wilcoxon-based version achieves higher accuracy than a DESeq-based version. Summarize each scenario's p-value samples by their medians p_j^m(k) and 2.5-97.5% quantile intervals, storing one p-value matrix per scenario. These p-values feed downstream steps: plot_bin_pv for -log10 median p-value traces, plot_bin_psig for the significant-feature U-curve, and bin_fragility for the fragility index FI_j = min_k(p^m_j(k) > 0.05).

## Related tools

- **ProgPermute** (R package providing bin_permute_all and bin_multitest that perform progressive label exchange and per-feature Wilcoxon rank sum testing in each scenario) — github.com/LyonsZhang/ProgPermute
- **R** (Implementation language; testdata1 is loaded in R and p-value matrices are saved as permutation_pvalues.rds)
- **R-shiny** (User-friendly interface exposing the progressive permutation and Wilcoxon-based testing workflow with visualizations)
- **DESeq** (Alternative p-value source that can replace the Wilcoxon default, though with lower accuracy on simulations)

## Examples

```
results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)
```

## Evaluation signals

- Each scenario's p-value matrix contains p rows (one per feature, p=267 for testdata1) with defined median p-values for every scenario k
- Traces of significant variables in plot_bin_pv output start high at k=0 (no permutation) and decrease toward nonsignificance as mixing approaches the full permutation scenario
- Number of significant features nsig(k) is lowest at the full permutation scenario Kf where p-values approach 1
- Per-feature fragility index satisfies 0 <= FI_j < K_f, where FI_j = min_k(p^m_j(k) > 0.05)
- Wilcoxon-based version outperforms the DESeq-based version in simulated-data accuracy comparisons

## Limitations

- The method does not control the false discovery rate or calculate adjusted p-values; empirical Bayes FDR adjustment is only planned future work
- Measures such as AOI or AUMC derived from these p-values are meant to be descriptive, not formal inference
- Ordering of signals by fragility index is not perfect, though simulations show high accuracy for identifying differential features

## Evidence

- [abstract] Default nonparametric test choice: "By default, we use the Wilcoxon rank sum test to compute the p-values, since it is a robust nonparametric test."
- [abstract] Per-scenario multiple testing: "we progressively permute the grouping factor labels of the microbiome samples and perform multiple differential abundance tests in each scenario."
- [methods] Wilcoxon beats DESeq on simulations: "the version based on the Wilcoxon test achieves higher accuracy than the version based on DESeq"
- [readme] Concrete ProgPermute invocation: "results<-bin_permute_all(variable="variable",testdata=testdata1,top_pm=267,zoomn=15,alpha=0.05)"
