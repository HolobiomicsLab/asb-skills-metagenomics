---
name: heterogeneity-variance-estimation-sj
description: Use when combining multiple studies across multiple related markers (e.g.,
  6 α-diversity indices across 15 HIV microbiome studies) from study-by-marker effect
  estimates and standard errors, requiring a random effects heterogeneity variance
  estimator with no prior marker-by-marker correlations.
when_to_use_negative:
- 'Fewer than 10 studies: mMeta.aMeta requires the number of studies (rows in ''est''/''std.err'')
  to be ≥10'
- 'Input is raw sequence data or an OTU/feature table: the method requires pre-computed
  summary statistics (effect estimates and their standard errors), which the discussion
  notes ''are still hard to obtain in practice'''
- Studies report only stratified summaries (mean, median, IQR per group), graphs,
  or significance categories rather than effect estimates with standard errors — the
  article notes this makes conventional meta-analysis challenging
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mMeta
  role: R package providing the mMeta.aMeta function that performs random effects
    heterogeneity variance estimation, single-marker and multi-marker (mMeta) meta-analysis,
    and the adaptive aMeta test
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment (R >= 4.0.3) in which the mMeta package is installed via
    devtools and executed
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:0c9c0c392d81d060d613b05edb97b696bfe34e7a759c27348780f010b678502a
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# heterogeneity-variance-estimation-sj

## Summary

Random effects heterogeneity variance estimation for multi-study, multi-marker meta-analysis using summary statistics (regression coefficients and standard errors), as implemented in the mMeta R package. It supports the SJ heterogeneity estimator together with Han and Eskin's modification (tau0=FALSE) or the traditional random effects null (tau0=TRUE).

## When to use

Use when combining multiple studies across multiple related markers (e.g., 6 α-diversity indices across 15 HIV microbiome studies) from study-by-marker effect estimates and standard errors, requiring a random effects heterogeneity variance estimator with no prior marker-by-marker correlations.

## When NOT to use

- Fewer than 10 studies: mMeta.aMeta requires the number of studies (rows in 'est'/'std.err') to be ≥10
- Input is raw sequence data or an OTU/feature table: the method requires pre-computed summary statistics (effect estimates and their standard errors), which the discussion notes 'are still hard to obtain in practice'
- Studies report only stratified summaries (mean, median, IQR per group), graphs, or significance categories rather than effect estimates with standard errors — the article notes this makes conventional meta-analysis challenging

## Inputs

- data.frame or matrix of study-specific effect estimates (e.g., regression coefficients), rows = studies, columns = markers, NAs allowed
- data.frame or matrix of study-specific standard errors with NA positions matched to 'est'
- Summary data shipped with the package: HIV.Sum.Data (15 HIV studies × 6 α-diversity indices: Richness (Observed), Shannon, Simpson, PD, PE, PQE) or Sum.Data

## Outputs

- Pooled estimates and 95% confidence intervals for each marker
- Marker-specific p-values from single-marker random effects meta-analyses
- mMeta p-value across markers (multi-marker meta-analysis)
- aMeta p-value across markers (adaptive minimum-p-value test via group permutation null distribution)

## How to apply

Assemble two matched data.frames: 'est' (study-specific regression coefficients) and 'std.err' (study-specific standard errors), with rows as studies and columns as markers; NA positions must match between the two, and the number of studies must be ≥10. Call mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123) for the Han and Eskin modified random effects meta-analysis (heterogeneity variance assumed to not exist under the null), or tau0=TRUE for the traditional random effects null; the task cards specify the SJ heterogeneity variance estimator. Use n.perm=5000 group permutations; if the number of studies <15, the function automatically enumerates all possible permutations and ignores n.perm and seed. Fix seed=123 for reproducibility and extract the pooled estimates, 95% confidence intervals, single-marker p-values, mMeta p-value across markers, and aMeta p-value from the returned object.

## Related tools

- **mMeta** (R package providing the mMeta.aMeta function that performs random effects heterogeneity variance estimation, single-marker and multi-marker (mMeta) meta-analysis, and the adaptive aMeta test) — https://github.com/hk1785/mMeta
- **R** (Runtime environment (R >= 4.0.3) in which the mMeta package is installed via devtools and executed) — https://github.com/hk1785/mMeta

## Examples

```
library(mMeta); data(HIV.Sum.Data); out2 <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Rerun with seed=123: the marker-specific p-values and the aMeta p-value must reproduce identically across repeated calls
- The number of studies (rows of 'est'/'std.err') is ≥10 and NA positions match exactly between 'est' and 'std.err'
- Refitting with tau0=TRUE versus tau0=FALSE on the same data yields different p-value sets; the Han-Eskin modification (tau0=FALSE) is expected to produce smaller p-values (Figure 2, Figure 3, Table S2 comparison)
- If the number of studies <15, the function ignores n.perm and seed and the result is deterministic (all possible permutations enumerated)
- Output contains pooled estimates, 95% CIs, per-marker p-values, an mMeta p-value across markers, and an aMeta p-value, matching the documented 'Values' of mMeta.aMeta

## Limitations

- Requires only effect estimates and their standard errors, but the discussion notes these 'are still hard to obtain in practice' — many studies report only graphs or stratified summaries
- The estimator is limited to hypothesis testing for aMeta; mMeta has only moderate power
- No a priori marker-by-marker correlations are used; correlations are estimated non-parametrically via permutations
- No version number stated for the mMeta R package and no changelog found for the repository
- Missing summary data are handled, but NA locations in 'est' and 'std.err' must match

## Evidence

- [other] Han and Eskin's modification (tau0=FALSE) vs. the traditional random effects null (tau0=TRUE) with the SJ estimator: [evidence span withheld — anchor status: not_found]
