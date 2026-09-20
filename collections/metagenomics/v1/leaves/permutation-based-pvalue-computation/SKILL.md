---
name: permutation-based-pvalue-computation
description: Use when running mMeta.aMeta on study-by-marker summary statistics (regression
  coefficient estimates and standard errors) and hypothesis-testing p-values are needed
  across multiple related markers, particularly when marker-by-marker correlations
  are unknown.
when_to_use_negative:
- You need a pooled effect estimate rather than hypothesis testing — aMeta's permutation-based
  across-marker p-value is limited to hypothesis testing, while mMeta's power is only
  moderate
- You have fewer than 10 studies per the mMeta.aMeta argument specification (the number
  of studies, i.e., the number of rows, should be greater than or equal to 10)
- Marker-by-marker correlations are known a priori, which would make a conventional
  covariance-aware meta-analysis preferable to the permutation-based correlation estimation
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mMeta
  role: R package implementing the sign-flipping group permutation test (n.perm=5000)
    that supplies the permutation-based p-values for mMeta and aMeta
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment for the mMeta R package (requires R >= 4.0.3)
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:8f520028738a3c5b467e214e2d2f201d945cb7f933b7cc424c0f88786de3cf4b
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# permutation-based-pvalue-computation

## Summary

Computes p-values for mMeta and aMeta multi-marker meta-analysis using sign-flipping group permutation with n.perm=5000 instead of asymptotic normality, because the group permutation method is more powerful for testing whether multiple related markers (e.g., six α-diversity indices across 15 HIV microbiome studies) are jointly associated with the trait.

## When to use

Use when running mMeta.aMeta on study-by-marker summary statistics (regression coefficient estimates and standard errors) and hypothesis-testing p-values are needed across multiple related markers, particularly when marker-by-marker correlations are unknown.

## When NOT to use

- You need a pooled effect estimate rather than hypothesis testing — aMeta's permutation-based across-marker p-value is limited to hypothesis testing, while mMeta's power is only moderate
- You have fewer than 10 studies per the mMeta.aMeta argument specification (the number of studies, i.e., the number of rows, should be greater than or equal to 10)
- Marker-by-marker correlations are known a priori, which would make a conventional covariance-aware meta-analysis preferable to the permutation-based correlation estimation

## Inputs

- est: data.frame or matrix of study-specific regression coefficient estimates, rows = studies, columns = markers (NAs allowed, matched between est and std.err)
- std.err: data.frame or matrix of study-specific standard errors, same dimensions as est
- n.perm: number of permutations (default 5000)
- seed: seed number for reproducible permutation outcomes (e.g., 123)

## Outputs

- p-values for each marker from the single-marker random effects meta-analysis
- across-marker p-value using mMeta
- across-marker p-value using aMeta (minimum-p-value test)
- pooled estimates and 95% confidence intervals for each marker and across markers

## How to apply

Call mMeta.aMeta(est, std.err, seed=123) from the R package mMeta (github.com/hk1785/mMeta) with the study-by-marker 'est' and 'std.err' matrices; the function internally applies the single-marker random effects meta-analysis (SJ heterogeneity estimator) with sign-flipping group permutation at the default n.perm=5000 to derive p-values, including the aMeta minimum-p-value across-marker test. Set seed explicitly (e.g., seed=123) to obtain the same outcomes repeatedly, since permutation results are stochastic. The group permutation approach is chosen over asymptotic normality because it is empirically more powerful for the multi-marker test. If the number of studies is fewer than 15, all possible permutations are implemented and n.perm and seed are ignored, which removes stochasticity entirely. Extract the marker-specific p-values and the across-marker mMeta and aMeta p-values from the output object for downstream reporting (e.g., writing to a CSV results table).

## Related tools

- **mMeta** (R package implementing the sign-flipping group permutation test (n.perm=5000) that supplies the permutation-based p-values for mMeta and aMeta) — https://github.com/hk1785/mMeta
- **R** (Runtime environment for the mMeta R package (requires R >= 4.0.3)) — https://github.com/hk1785/mMeta

## Examples

```
out2 <- mMeta.aMeta(est, std.err, seed=123)
```

## Evaluation signals

- Setting a seed (e.g., seed=123) reproduces the same p-values on repeated mMeta.aMeta runs with the same inputs
- With 15 HIV studies (>= 15 rows), permutation uses n.perm=5000; with fewer than 15 studies, all possible permutations are run and results are deterministic
- The aMeta across-marker p-value should be smaller than the mMeta across-marker p-value in the real-data application, consistent with aMeta's higher power
- Computed p-values should match the values reported in Figure 3 and Table S2 of the paper (Koh et al. 2021, Statistics in Medicine 40(12), 2859-2879)
- mMeta.aMeta completes without error even when est/std.err contain unmatched-count NAs (missing summary data), and all returned p-values are non-empty

## Limitations

- Permutation is stochastic unless a seed is set; with fewer than 15 studies n.perm and seed are ignored because all possible permutations are implemented
- aMeta's permutation-based test is limited to hypothesis testing and provides no pooled effect estimate
- mMeta estimates marker-by-marker correlations non-parametrically via permutations, yet its power is only moderate
- The est matrix must have at least 10 studies (rows), and NA locations must match between est and std.err

## Evidence
