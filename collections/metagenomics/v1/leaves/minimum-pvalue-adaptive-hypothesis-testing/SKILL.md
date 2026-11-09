---
name: minimum-pvalue-adaptive-hypothesis-testing
description: Use when combining multiple studies (≥10) across multiple related markers
  (e.g., six α-diversity indices) from summary statistics alone, with no prior marker-by-marker
  correlations, and the goal is a single omnibus hypothesis test rather than a pooled
  effect estimate.
when_to_use_negative:
- You need a pooled effect estimate with 95% CI across all markers — use mMeta's weighted
  pooling instead, since aMeta is limited to hypothesis testing
- 'Fewer than 10 studies: the package requires ≥10 studies for mMeta.aMeta, and with
  <15 studies all possible permutations run and n.perm/seed are ignored'
- Only raw sequence data are available and effect estimates/standard errors have not
  been derived (the article notes these summary statistics are hard to obtain in practice)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0625
tools:
- name: mMeta
  role: R package providing mMeta.aMeta, which computes the marker-specific random-effects
    meta-analyses and the aMeta minimum-p-value test with sign-flipping group permutation
  repo: https://github.com/hk1785/mMeta
- name: R
  role: runtime environment (≥4.0.3) for installing and running the mMeta package
- name: forestplot
  role: dependency used by mMeta.aMeta.plot to draw forest plots of per-marker and
    across-marker results
- name: MASS
  role: imported dependency of the mMeta package
- name: vegan
  role: imported dependency of the mMeta package (α-diversity computation context)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:f67d3cfe56a16e48ced3d18c2498edcbd0f8eb05ffa268cd5b679c4d97697e68
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - MASS
  - R
  - R package mMeta
  - forestplot
  - mMeta
  - vegan
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# minimum-pvalue-adaptive-hypothesis-testing

## Summary

aMeta (adaptive multi-marker meta-analysis) tests the overall null hypothesis across multiple related markers using only study-specific effect estimates and standard errors, via the minimum p-value among marker-specific random-effects meta-analyses calibrated by sign-flipping group permutation; it delivers near-maximal power for hypothesis testing but yields no pooled effect estimate.

## When to use

Use when combining multiple studies (≥10) across multiple related markers (e.g., six α-diversity indices) from summary statistics alone, with no prior marker-by-marker correlations, and the goal is a single omnibus hypothesis test rather than a pooled effect estimate.

## When NOT to use

- You need a pooled effect estimate with 95% CI across all markers — use mMeta's weighted pooling instead, since aMeta is limited to hypothesis testing
- Fewer than 10 studies: the package requires ≥10 studies for mMeta.aMeta, and with <15 studies all possible permutations run and n.perm/seed are ignored
- Only raw sequence data are available and effect estimates/standard errors have not been derived (the article notes these summary statistics are hard to obtain in practice)

## Inputs

- data.frame or matrix 'est' of study-specific estimates (regression coefficients), rows = studies (≥10, named), columns = markers, NAs allowed but matched to std.err
- data.frame or matrix 'std.err' of study-specific standard errors with identical row/column names and matched NA locations
- summary dataset HIV.Sum.Data (15 HIV microbiome studies × 6 α-diversity indices: Richness (Observed), Shannon, Simpson, PD, PE, PQE) shipped with the mMeta package

## Outputs

- aMeta across-marker p-value (minimum p-value test over marker-specific random-effects meta-analyses, permutation-calibrated)
- per-marker pooled estimates, 95% confidence intervals and p-values from the marker-specific random-effects meta-analyses
- forest plot across markers via mMeta.aMeta.plot
- results table (e.g., mmeta_ameta_hiv_results.csv) comparable to Figure 3 and Table S2 of the paper

## How to apply

Assemble a matrix of study-specific regression coefficients (est) and matching standard errors (std.err), with rows as studies (≥10) and columns as markers, where NA locations match between the two matrices. Run mMeta.aMeta(est, std.err, seed=123), which computes marker-specific random-effects meta-analyses (SJ heterogeneity estimator) and then the aMeta minimum-p-value test, calibrated by sign-flipping group permutation with n.perm=5000 (all possible permutations are used, and n.perm/seed are ignored, if the number of studies < 15). Extract the aMeta across-marker p-value and compare it to the mMeta across-marker p-value; in the 15-study HIV microbiome application across Richness, Shannon, Simpson, PD, PE and PQE, the aMeta p-value should be smaller, reflecting its higher power. Optionally visualize per-marker and across-marker results with mMeta.aMeta.plot. Note aMeta is limited to hypothesis testing and does not provide a pooled estimate or confidence interval across markers; use mMeta when an overall estimate/CI is needed.

## Related tools

- **mMeta** (R package providing mMeta.aMeta, which computes the marker-specific random-effects meta-analyses and the aMeta minimum-p-value test with sign-flipping group permutation) — https://github.com/hk1785/mMeta
- **R** (runtime environment (≥4.0.3) for installing and running the mMeta package)
- **forestplot** (dependency used by mMeta.aMeta.plot to draw forest plots of per-marker and across-marker results)
- **MASS** (imported dependency of the mMeta package)
- **vegan** (imported dependency of the mMeta package (α-diversity computation context))

## Examples

```
library(mMeta); data(HIV.Sum.Data); est <- HIV.Sum.Data$est; std.err <- HIV.Sum.Data$std.err; out2 <- mMeta.aMeta(est, std.err, seed=123)
```

## Evaluation signals

- Reproduce the 15-study HIV application: with seed=123 and n.perm=5000, the aMeta across-marker p-value must be smaller than the mMeta across-marker p-value, matching Figure 3 and Table S2 of the paper
- Row and column names of 'est' and 'std.err' must match, and NA locations must coincide between the two matrices
- Number of studies (rows) must be ≥10; otherwise the function's assumptions are violated
- With ≥15 studies the permutation seed is honored (repeatable p-values with seed=123); with <15 studies all possible permutations run and results are seed-independent
- Per-marker pooled estimates/CIs/p-values from the same call should agree with the marker-specific random-effects meta-analyses reported in the paper

## Limitations

- aMeta is limited to hypothesis testing — it yields an across-marker p-value but no pooled estimate or confidence interval
- Requires only effect estimates and their standard errors, but these summary statistics are hard to obtain in practice; many studies report only graphs (box plots, bar/line charts with error bars) or stratified group summaries rather than association estimates
- With <15 studies, all possible permutations are implemented and n.perm/seed are ignored
- There are no set criteria for selecting candidate markers; selection rules should be documented so readers can judge their reasonableness
- Heterogeneous variance under the null is controlled by tau0 (traditional random effects vs. Han–Eskin's modified random effects); the default tau0=FALSE corresponds to Han–Eskin

## Evidence

- [methods] real-data application of aMeta: [evidence span withheld — anchor status: not_found]
