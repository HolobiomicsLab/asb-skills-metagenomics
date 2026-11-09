---
name: group-permutation-null-distribution-generation
description: Use when performing adaptive multi-marker meta-analysis (aMeta) on effect
  estimates and standard errors from multiple related markers across studies, and
  a null distribution for the minimum p-value statistic must be generated without
  prior marker-by-marker correlations.
when_to_use_negative:
- Fewer than 10 studies (rows) per marker — the mMeta.aMeta function requires ≥10
  studies
- Only a single marker is tested — the minimum-p-value statistic and its group-permutation
  null distribution are specific to multiple related markers
- Raw sequence data rather than study-by-marker summary statistics — mMeta and aMeta
  require only effect estimates and their standard errors, and such estimates are
  often hard to obtain from studies reporting graphs or stratified summaries
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: mMeta
  role: R package providing the mMeta.aMeta function that implements the group permutation
    null distribution for mMeta and aMeta (n.perm=5000, tau0 flag, seed argument)
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment for the mMeta R package (requires R >= 4.0.3, with devtools
    used to install from GitHub)
  repo: https://github.com/hk1785/mMeta
- name: devtools
  role: Prerequisite R package used to install mMeta from GitHub via install_github("hk1785/mMeta",
    force=T)
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:aae3a57207c80e06244e055e31b5c2dd6bb5909dd495720787b2b787fc1f4a92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - devtools
  - mMeta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# group-permutation-null-distribution-generation

## Summary

Generates the null distribution for the aMeta minimum-p-value test statistic by applying the Follmann and Proschan (1999) group permutation method to study-specific effect estimates, avoiding any a priori modeling of marker-by-marker correlations across related markers (e.g., six α-diversity indices).

## When to use

Use when performing adaptive multi-marker meta-analysis (aMeta) on effect estimates and standard errors from multiple related markers across studies, and a null distribution for the minimum p-value statistic must be generated without prior marker-by-marker correlations.

## When NOT to use

- Fewer than 10 studies (rows) per marker — the mMeta.aMeta function requires ≥10 studies
- Only a single marker is tested — the minimum-p-value statistic and its group-permutation null distribution are specific to multiple related markers
- Raw sequence data rather than study-by-marker summary statistics — mMeta and aMeta require only effect estimates and their standard errors, and such estimates are often hard to obtain from studies reporting graphs or stratified summaries

## Inputs

- est: data.frame or matrix of study-specific effect estimates (e.g., regression coefficients), rows = studies, columns = markers, NAs allowed if matched between est and std.err, ≥10 studies
- std.err: data.frame or matrix of study-specific standard errors, rows = studies, columns = markers, NAs matched to est, ≥10 studies

## Outputs

- Marker-specific mMeta pooled estimates, 95% confidence intervals, and p-values for each marker
- mMeta p-value across markers
- aMeta p-value across markers derived from the group-permutation null distribution of the minimum p-value statistic

## How to apply

Load study-by-marker effect estimates and standard errors (e.g., Sum.Data or HIV.Sum.Data: 15 HIV studies × 6 α-diversity indices: Richness (Observed), Shannon, Simpson, PD, PE, PQE) and run mMeta.aMeta(est, std.err) from the mMeta R package. For each marker, first fit a single-marker random effects meta-analysis using the SJ heterogeneity variance estimator, requiring at least K = 10 studies per marker. Then apply the group permutation method by randomly assigning ±1 signs to the absolute effect estimates; by default R = 5000 permutations are used (n.perm=5000), but if the number of studies is less than 15, all 2^K possible permutations are enumerated and n.perm and seed are ignored. Recompute null pooled effects with heterogeneity estimates set to zero under the Han and Eskin modification (tau0=FALSE), derive marker-specific null p-values, and form the aMeta statistic T = min Pj; its p-value is the proportion of permutation-level minimum p-values equal to or more extreme than the observed minimum. Use seed=123 for reproducibility, and validate by comparing marker-specific and aMeta p-values against the mMeta.aMeta(seed=123) package output.

## Related tools

- **mMeta** (R package providing the mMeta.aMeta function that implements the group permutation null distribution for mMeta and aMeta (n.perm=5000, tau0 flag, seed argument)) — https://github.com/hk1785/mMeta
- **R** (Runtime environment for the mMeta R package (requires R >= 4.0.3, with devtools used to install from GitHub)) — https://github.com/hk1785/mMeta
- **devtools** (Prerequisite R package used to install mMeta from GitHub via install_github("hk1785/mMeta", force=T))

## Examples

```
library(mMeta); data(HIV.Sum.Data); out2 <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, seed=123)
```

## Evaluation signals

- Reimplemented marker-specific p-values match mMeta.aMeta(seed=123) package output on the same example data (Sum.Data or HIV.Sum.Data)
- The overall aMeta p-value matches the package output within permutation-seed reproducibility
- With ≥15 studies, exactly n.perm=5000 permutations are used; with <15 studies, all 2^K possible permutations are enumerated and the seed is ignored
- Locations of NAs in est and std.err are matched and the number of rows (studies) is ≥10 before fitting
- Running with seed=123 reproduces identical p-values across repeated calls (deterministic behavior when a seed is supplied)

## Limitations

- Estimating marker-by-marker correlations non-parametrically via permutations yields only moderate power for mMeta; aMeta is more powerful but limited to hypothesis testing (no pooled estimates)
- Effect estimates and their standard errors required for implementation are hard to obtain in practice; many studies report only graphs or stratified summary statistics rather than association estimates
- No set criteria exist for selecting candidate markers for mMeta and aMeta, which affects which markers enter the minimum-p-value statistic
- No version number stated for the mMeta R package and no changelog found in the repository

## Evidence
