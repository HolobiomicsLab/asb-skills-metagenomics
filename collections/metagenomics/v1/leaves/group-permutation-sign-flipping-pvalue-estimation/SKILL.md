---
name: group-permutation-sign-flipping-pvalue-estimation
description: Use when combining multiple studies (>=10) across related markers from
  effect estimates and standard errors only, with no a priori marker-by-marker correlations,
  and a semi-parametric null is preferred over parametric asymptotic normality.
when_to_use_negative:
- 'Fewer than 10 studies: the mMeta.aMeta function requires the number of studies
  (rows of est/std.err) to be greater than or equal to 10'
- 'Raw sequence/OTU count data are the input: mMeta and aMeta require pre-computed
  effect estimates and their standard errors, not count tables'
- A fully parametric inference based on asymptotic normality of mu_hat/sigma_mu is
  specifically required by the analysis plan (then compare, but the permutation method
  is not the target)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: mMeta
  role: R package providing mMeta.aMeta (group permutation estimation of marker-by-marker
    correlations and aMeta null statistics) and the HIV.Sum.Data/Sum.Data summary
    datasets
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment for the mMeta package (requires R >= 4.0.3, with devtools,
    forestplot, MASS, vegan)
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:718e2adb98bffa28ff4cad38a7ba5626ccc4356794f64982148dab6a4e67f739
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# group-permutation-sign-flipping-pvalue-estimation

## Summary

Semi-parametric p-value calculation for mMeta/aMeta in which marker-by-marker correlations and null distributions are estimated via permutations/sign-flipping of study-level summary statistics, avoiding reliance on asymptotic normality of mu_hat/sigma_mu ~ N(0,1). Used because the group permutation method is more powerful (yields smaller p-values) than the parametric asymptotic-normality method.

## When to use

Use when combining multiple studies (>=10) across related markers from effect estimates and standard errors only, with no a priori marker-by-marker correlations, and a semi-parametric null is preferred over parametric asymptotic normality.

## When NOT to use

- Fewer than 10 studies: the mMeta.aMeta function requires the number of studies (rows of est/std.err) to be greater than or equal to 10
- Raw sequence/OTU count data are the input: mMeta and aMeta require pre-computed effect estimates and their standard errors, not count tables
- A fully parametric inference based on asymptotic normality of mu_hat/sigma_mu is specifically required by the analysis plan (then compare, but the permutation method is not the target)

## Inputs

- data.frame/matrix of study-specific effect estimates (est), rows = studies, columns = markers, NAs allowed with matched NA positions
- data.frame/matrix of study-specific standard errors (std.err), same shape and NA positions as est
- packaged example datasets Sum.Data and HIV.Sum.Data (effect estimates and SEs for 15 HIV studies across 6 alpha-diversity indices: Richness, Shannon, Simpson, PD, PE, PQE)

## Outputs

- Pooled estimates, 95% confidence intervals, and p-values for each marker and across markers using mMeta
- A p-value across markers using aMeta (minimum p-value statistic with permutation-based null)
- Side-by-side comparison table of permutation vs. parametric p-values and 5%-level power estimates (e.g., pvalue_method_comparison.csv)

## How to apply

Install the mMeta package from GitHub with devtools (install_github('hk1785/mMeta', force=T)) and load summary data as a data.frame of study-specific estimates (est) and standard errors (std.err), with NAs matched between the two matrices. Call mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123): mMeta uses the group permutation method to estimate marker-by-marker correlations, and aMeta uses it to generate null statistic values of the minimum p-value statistic across markers. Set tau0=FALSE for the Han-Eskin modification assuming no heterogeneous variance under the null (traditional random-effects is tau0=TRUE); note that when the number of studies is <15, all possible permutations are implemented and n.perm/seed are ignored. For comparison, also run the parametric method that assumes mu_hat/sigma_mu follows N(0,1) for each marker, then tabulate p-values side by side — the group permutation method yields smaller p-values and higher power at the 5% significance level (Figure 1, Table S2). Power can additionally be estimated in silico with Dirichlet-multinomial count tables from 100 respiratory-tract OTUs under scenarios S1-S4 (rare, random, common, PAM phylogenetic cluster) for K=10, 20 or 30 studies.

## Related tools

- **mMeta** (R package providing mMeta.aMeta (group permutation estimation of marker-by-marker correlations and aMeta null statistics) and the HIV.Sum.Data/Sum.Data summary datasets) — https://github.com/hk1785/mMeta
- **R** (Runtime environment for the mMeta package (requires R >= 4.0.3, with devtools, forestplot, MASS, vegan)) — https://github.com/hk1785/mMeta

## Examples

```
library(devtools); install_github("hk1785/mMeta", force=T); library(mMeta); data(HIV.Sum.Data); out <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Group permutation p-values should be smaller than (i.e., more powerful than) the parametric asymptotic-normality p-values, matching the ordering reported in Figure 1 and Table S2
- Setting seed=123 must reproduce the same p-values on repeated runs; with <15 studies all possible permutations are implemented and n.perm/seed are ignored (results deterministic)
- NA positions in est and std.err must match exactly, otherwise the call is invalid
- Estimated permutation power at the 5% significance level in simulated scenarios (K=10/20/30, scenarios S1-S4) should reflect the reported higher power of the group permutation method relative to asymptotic normality
- Output must contain pooled estimates, 95% CIs, per-marker and cross-marker mMeta p-values, and the cross-marker aMeta p-value

## Limitations

- Power of mMeta is only moderate when marker-by-marker correlations are estimated non-parametrically via permutations
- aMeta is limited to hypothesis testing — it provides a cross-marker p-value but no pooled effect estimate across markers
- The number of studies must be >= 10; when < 15, all possible permutations are used and n.perm/seed are ignored
- Effect estimates and their standard errors required by mMeta and aMeta are hard to obtain in practice, as many studies report only graphs or stratified summaries
- No version number stated for the mMeta R package and no changelog found for the repository

## Evidence

- [supplementary] mMeta uses it to estimate marker-by-marker correlations, and aMeta uses it to generate the null statistic values of the minimum p-value statistic.: [evidence span withheld — anchor status: not_found]
