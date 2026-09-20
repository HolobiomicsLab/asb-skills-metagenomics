---
name: permutation-based-meta-analysis-pvalue-estimation
description: Use when combining ≥10 studies across multiple related markers (e.g.,
  6 α-diversity indices) using only effect estimates and standard errors, with no
  a priori marker-by-marker correlations, and p-values must be obtained via permutations.
when_to_use_negative:
- Fewer than 10 studies are available — the mMeta.aMeta function requires the number
  of studies (rows) to be greater than or equal to 10
- Only raw sequencing count data (e.g., OTU tables) are available and study-specific
  effect estimates and standard errors have not yet been computed via regression —
  mMeta and aMeta require only the effect estimates and their standard errors
- A priori marker-by-marker correlations are already known, in which case conventional
  multivariate meta-analysis methods that use the correlation matrix directly are
  more appropriate
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3941
tools:
- name: R package mMeta
  role: Provides the mMeta.aMeta function that conducts the group-permutation-based
    marker-specific (mMeta) and multi-marker (aMeta) meta-analyses on summary statistics
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment (R >= 4.0.3) in which the mMeta package is installed via
    devtools and the permutation meta-analyses are executed
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:3a2045a4e1f28c34a72b1bb5331e7d35eb556d1e2f1f6b5bffbdceee86045e59
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# permutation-based-meta-analysis-pvalue-estimation

## Summary

Estimate marker-specific and multi-marker meta-analysis p-values by non-parametrically permuting study labels via the group permutation method, as implemented in the mMeta R package's mMeta.aMeta function for combining multiple studies across multiple related markers using only summary statistics (effect estimates and standard errors).

## When to use

Use when combining ≥10 studies across multiple related markers (e.g., 6 α-diversity indices) using only effect estimates and standard errors, with no a priori marker-by-marker correlations, and p-values must be obtained via permutations.

## When NOT to use

- Fewer than 10 studies are available — the mMeta.aMeta function requires the number of studies (rows) to be greater than or equal to 10
- Only raw sequencing count data (e.g., OTU tables) are available and study-specific effect estimates and standard errors have not yet been computed via regression — mMeta and aMeta require only the effect estimates and their standard errors
- A priori marker-by-marker correlations are already known, in which case conventional multivariate meta-analysis methods that use the correlation matrix directly are more appropriate

## Inputs

- est: data.frame (or matrix) of study-specific estimates (e.g., regression coefficients), rows = studies, columns = markers, may contain NAs matched to std.err
- std.err: data.frame (or matrix) of study-specific standard errors with NAs matching those in est
- n.perm: number of permutations (default 5000)
- seed: seed number for reproducibility (default NULL)
- tau0: indicator of whether heterogeneous variance exists under the null (TRUE = traditional random effects, FALSE = Han-Eskin modification)

## Outputs

- Pooled estimates for each marker and across markers
- 95% confidence intervals for each marker and across markers
- Marker-specific p-values (mMeta)
- Multi-marker p-value across markers using mMeta
- Multi-marker p-value across markers using aMeta

## How to apply

Install the mMeta R package from GitHub (install_github('hk1785/mMeta', force=T)) and load summary data such as HIV.Sum.Data (15 HIV studies x 6 alpha-diversity indices: Richness/Observed, Shannon, Simpson, PD, PE, PQE). Call mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123) on data.frames of study-specific estimates and standard errors (rows = studies, columns = markers; NAs allowed but must match between est and std.err). Choose tau0=FALSE for the Han-Eskin modified random effects meta-analysis (heterogeneity variance does not exist under the null) or tau0=TRUE for the traditional random effects meta-analysis (heterogeneity variance exists under the null); the Han-Eskin modification yields smaller p-values. The heterogeneity variance is estimated with the Sidik-Jonkman (SJ) robust/sandwich estimator, which gives a robust estimate based on a refinement using an initial heterogeneity variance estimate. The input must have ≥10 studies; if the number of studies < 15, all possible permutations are implemented and n.perm/seed are ignored. The group permutation method is preferred over the asymptotic normality method because it is more powerful (Figure 1A,B < Figure 1C,D).

## Related tools

- **R package mMeta** (Provides the mMeta.aMeta function that conducts the group-permutation-based marker-specific (mMeta) and multi-marker (aMeta) meta-analyses on summary statistics) — https://github.com/hk1785/mMeta
- **R** (Runtime environment (R >= 4.0.3) in which the mMeta package is installed via devtools and the permutation meta-analyses are executed) — https://github.com/hk1785/mMeta

## Examples

```
out2 <- mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123); out2
```

## Evaluation signals

- Re-running mMeta.aMeta with the same seed (e.g., seed=123) reproduces identical p-values; if the number of studies < 15, results must be identical because all possible permutations are implemented and n.perm is ignored
- The number of studies (rows of est and std.err) must be ≥ 10, and NA locations must match between est and std.err
- Group permutation p-values should be at least as small (more powerful) than those from the asymptotic normality method, consistent with Figure 1A,B < Figure 1C,D
- Han-Eskin modified analyses (tau0=FALSE) should produce smaller p-values than the traditional random effects analyses (tau0=TRUE), as in Figure 3B vs Figure 3A and Table S2
- Type I error rates from permutation-based analyses should be well-controlled at <= 5% across simulation scenarios

## Limitations

- mMeta's power is only moderate because it estimates marker-by-marker correlations non-parametrically via permutations; aMeta closely approaches the highest power but is limited to hypothesis testing
- The required effect estimates and their standard errors are hard to obtain in practice — many of the 15 microbiome studies reported only graphs (e.g., box plots, bar/line charts with error bars) with no numbers or stratified effect estimates rather than association estimates
- No set criteria exist for selecting candidate markers for mMeta and aMeta; selection rules should be explicitly delineated
- The package has no stated version numbering beyond Version 1.0 (2021-2-11) and no changelog; depends on R (>= 4.0.3) and imports forestplot, MASS, vegan

## Evidence

- [methods] data(HIV.Sum.Data) # Summary data for 15 HIV studies and 6 alpha-diversity indices: [evidence span withheld — anchor status: not_found]
