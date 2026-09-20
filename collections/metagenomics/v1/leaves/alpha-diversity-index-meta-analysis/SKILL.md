---
name: alpha-diversity-index-meta-analysis
description: Use when you have study-by-marker effect estimates and standard errors
  (no raw data, no a priori marker-by-marker correlations) for multiple related markers
  such as α-diversity indices and need an overall hypothesis test or pooled estimate.
when_to_use_negative:
- Raw sequence or OTU-table data is all you have — you must first obtain study-specific
  regression coefficients and standard errors; mMeta/aMeta require only summary statistics,
  which the paper notes are 'still hard to obtain in practice'
- Only a single marker (one α-diversity index) is of interest — then a conventional
  single-marker random effects meta-analysis suffices and the multi-marker machinery
  (mMeta/aMeta) is unnecessary
- Fewer than 10 studies are available — the README states 'The number of studies (i.e.,
  the number of rows) should be greater than or equal to 10'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mMeta
  role: R package providing mMeta.aMeta (multi-marker and adaptive multi-marker meta-analysis)
    and mMeta.aMeta.plot (forest plots); implements the SJ heterogeneity estimator,
    Han-Eskin modification (tau0=FALSE), and group permutation null (n.perm=5000,
    seed=123)
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Execution environment for the mMeta package; requires R >= 4.0.3 with forestplot,
    MASS and vegan imported
  repo: https://github.com/hk1785/mMeta
- name: R package mMeta
  role: Freely available package used to implement mMeta and aMeta using only summary
    statistics (effect estimates and standard errors)
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_002
  - task_006
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:16a74627fe86eed824c2426529b30916de92eb8ee8b3682d51e3258eb4b7c1a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# alpha-diversity-index-meta-analysis

## Summary

Meta-analysis skill that combines multiple microbiome studies across multiple related α-diversity markers (e.g., Richness, Shannon, Simpson, PD, PE, PQE) using only summary statistics — regression coefficients (est) and standard errors (std.err) — via the mMeta (pooled estimate) and aMeta (minimum-p-value test) methods from Koh & Zhao (2021, Statistics in Medicine).

## When to use

Use when you have study-by-marker effect estimates and standard errors (no raw data, no a priori marker-by-marker correlations) for multiple related markers such as α-diversity indices and need an overall hypothesis test or pooled estimate.

## When NOT to use

- Raw sequence or OTU-table data is all you have — you must first obtain study-specific regression coefficients and standard errors; mMeta/aMeta require only summary statistics, which the paper notes are 'still hard to obtain in practice'
- Only a single marker (one α-diversity index) is of interest — then a conventional single-marker random effects meta-analysis suffices and the multi-marker machinery (mMeta/aMeta) is unnecessary
- Fewer than 10 studies are available — the README states 'The number of studies (i.e., the number of rows) should be greater than or equal to 10'

## Inputs

- R data.frame/matrix 'est' of study-specific regression coefficients (rows = studies, columns = markers; NAs allowed at matched positions; >= 10 studies)
- R data.frame/matrix 'std.err' of study-specific standard errors (rows = studies, columns = markers; NAs must match 'est')
- Built-in example dataset HIV.Sum.Data (15 HIV studies x 6 alpha-diversity indices) or Sum.Data from the mMeta package

## Outputs

- Pooled estimates, 95% confidence intervals, and p-values for each marker
- mMeta across-marker pooled estimate, SE, 95% CI, and p-value
- aMeta across-marker p-value (minimum-p-value test over group permutations)
- Forest plot (via mMeta.aMeta.plot) showing per-marker and across-marker results

## How to apply

Install the mMeta R package from GitHub (install_github("hk1785/mMeta"), requires R >= 4.0.3 plus forestplot, MASS and vegan) and load the study-by-marker matrices, e.g. the built-in HIV.Sum.Data with 15 HIV studies x 6 alpha-diversity indices (Richness, Shannon, Simpson, PD, PE, PQE). Call mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123), which runs single-marker random effects meta-analysis with the SJ heterogeneity estimator, applies the Han-Eskin modification (tau0=FALSE, so heterogeneity variance is assumed not to exist under the null, avoiding the overly conservative traditional random effects null), and uses Follmann-Proschan sign-flipping group permutation (n.perm=5000, or all 2^K permutations when K < 15 studies) to generate the null distribution of the aMeta minimum-p-value statistic without prior marker-by-marker correlations. The output yields per-marker pooled estimates, 95% CIs and p-values, the mMeta across-marker pooled estimate/SE/CI/p-value (moderate power), and the aMeta across-marker p-value (closely approaches the highest power among marker-specific meta-analyses, but is limited to hypothesis testing). Missing study-by-marker entries are handled robustly: set NAs at matched positions in both est and std.err and the number of studies Kj varies by marker (minimum 10 studies per marker). Use mMeta.aMeta.plot(mMeta.aMeta.out=out) to draw forest plots per marker and across markers.

## Related tools

- **mMeta** (R package providing mMeta.aMeta (multi-marker and adaptive multi-marker meta-analysis) and mMeta.aMeta.plot (forest plots); implements the SJ heterogeneity estimator, Han-Eskin modification (tau0=FALSE), and group permutation null (n.perm=5000, seed=123)) — https://github.com/hk1785/mMeta
- **R** (Execution environment for the mMeta package; requires R >= 4.0.3 with forestplot, MASS and vegan imported) — https://github.com/hk1785/mMeta
- **R package mMeta** (Freely available package used to implement mMeta and aMeta using only summary statistics (effect estimates and standard errors)) — https://github.com/hk1785/mMeta

## Examples

```
library(devtools); install_github("hk1785/mMeta", force=T); data(HIV.Sum.Data); out2 <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, seed=123)
```

## Evaluation signals

- The mMeta.aMeta call runs with no error on unbalanced summary tables (matched NA positions in est and std.err) and outputs non-empty pooled estimates, CI bounds, and p-values for every marker plus the aMeta p-value
- With the same seed (seed=123), marker-specific p-values and the aMeta p-value are reproducible and match the mMeta.aMeta package output for the same data (permutation-seed reproducibility)
- The aMeta p-value is smaller than the mMeta p-value on the HIV data, confirming aMeta is the more powerful hypothesis-testing method (as reported for Figure 3 and Table S2)
- Refitting with tau0=TRUE (traditional random effects null) yields larger (more conservative) p-values than tau0=FALSE (Han-Eskin modification), matching the comparison reported for Figure 2, Figure 3, and Table S2
- Number of studies per marker is at least K = 10, and permutation count/seed follow n.perm=5000, seed=123 (or all 2^K permutations when K < 15)

## Limitations

- mMeta's power is only moderate, while aMeta closely approaches the highest power but is limited to hypothesis testing (no pooled estimate)
- Effect estimates and their standard errors required by mMeta and aMeta are hard to obtain in practice — many of the 15 microbiome studies reported only graphs (box plots, bar/line charts) with no numbers, or stratified effect estimates rather than association estimates
- NA positions in 'est' and 'std.err' must be matched; the traditional random effects null (tau0=TRUE) can be overly conservative compared with the Han-Eskin modification
- No set criteria exist for selecting candidate markers for mMeta and aMeta, and the six α-diversity indices are not always available from prior studies while other indices may be
- No version number is stated for the mMeta R package and no changelog is found for the repository; issues should be reported at https://github.com/hk1785/mMeta/issues

## Evidence

- [abstract] here we introduce two meta-analysis methods, multi-marker meta-analysis (mMeta) and adaptive multi-marker meta-analysis (aMeta), to combine multiple studies throughout multiple related markers: "here we introduce two meta-analysis methods, multi-marker meta-analysis (mMeta) and adaptive multi-marker meta-analysis (aMeta), to combine multiple studies throughout multiple related markers"
- [methods] data(HIV.Sum.Data) # Summary data for 15 HIV studies and 6 alpha-diversity indices: [evidence span withheld — anchor status: not_found]
- [intro] hence, we developed mMeta and aMeta to robustly handle missing summary data: "hence, we developed mMeta and aMeta to robustly handle missing summary data"
- [methods] the group permutation method is more powerful than the method based on the asymptotic normality: "the group permutation method is more powerful than the method based on the asymptotic normality"
- [readme] 'est' can contain missing elements (NAs), but the locations of NAs should be matched between 'est' and 'std.err': "'est' can contain missing elements (NAs), but the locations of NAs should be matched between 'est' and 'std.err'"
- [readme] The number of studies (i.e., the number of rows) should be greater than or equal to 10.: "The number of studies (i.e., the number of rows) should be greater than or equal to 10."
