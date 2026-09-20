---
name: multi-marker-summary-statistics-meta-analysis
description: Use when combining ≥10 studies across multiple related markers with only
  summary statistics (effect estimates and their standard errors, possibly with matched
  NAs), and no prior marker-by-marker correlations, e.g., α-diversity indices across
  microbiome studies.
when_to_use_negative:
- Input is raw sequencing data (e.g., FASTQ, OTU tables) rather than per-study effect
  estimates and standard errors — sequence processing (FLASH, Trimmomatic, PyNAST,
  Acacia, UCLUST, Bowtie2) and regression fitting must precede this skill
- Fewer than 10 studies are available, since mMeta.aMeta requires the number of studies
  (rows) to be greater than or equal to 10
- Only one marker needs meta-analysis — mMeta/aMeta are designed for combining multiple
  related markers, and aMeta is limited to hypothesis testing (no pooled estimate/CI)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2269
tools:
- name: mMeta
  role: R package providing mMeta.aMeta (multi-marker meta-analysis core function),
    mMeta.aMeta.plot (forest plots), and the HIV.Sum.Data example dataset
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment for mMeta and aMeta implementation (R >= 4.0.3; imports
    forestplot, MASS, vegan)
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:cae92a1005ad019bdf501d570ec477732fd1cb0221da1fe89ef31ea1140b99a5
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# multi-marker-summary-statistics-meta-analysis

## Summary

This skill implements mMeta and aMeta, two random effects meta-analysis methods that combine multiple studies across multiple related markers (e.g., six α-diversity indices) using only effect estimates (regression coefficients) and standard errors, with no a priori marker-by-marker correlations. mMeta estimates marker-by-marker correlations non-parametrically via group permutations for pooled estimates and tests, while aMeta adaptively weights single-marker tests and approaches the highest power among marker-specific meta-analyses for hypothesis testing.

## When to use

Use when combining ≥10 studies across multiple related markers with only summary statistics (effect estimates and their standard errors, possibly with matched NAs), and no prior marker-by-marker correlations, e.g., α-diversity indices across microbiome studies.

## When NOT to use

- Input is raw sequencing data (e.g., FASTQ, OTU tables) rather than per-study effect estimates and standard errors — sequence processing (FLASH, Trimmomatic, PyNAST, Acacia, UCLUST, Bowtie2) and regression fitting must precede this skill
- Fewer than 10 studies are available, since mMeta.aMeta requires the number of studies (rows) to be greater than or equal to 10
- Only one marker needs meta-analysis — mMeta/aMeta are designed for combining multiple related markers, and aMeta is limited to hypothesis testing (no pooled estimate/CI)

## Inputs

- data.frame or matrix 'est' of study-specific regression coefficients (rows = studies with study names, columns = markers with marker names; matched NAs allowed; ≥10 studies)
- data.frame or matrix 'std.err' of study-specific standard errors (same shape as 'est'; NA locations must match 'est')
- summary data such as HIV.Sum.Data from the mMeta package: regression coefficients and standard errors for 15 HIV studies × 6 α-diversity indices (Richness (Observed), Shannon, Simpson, PD, PE, PQE)

## Outputs

- Pooled estimates and 95% confidence intervals for each marker and across markers (mMeta)
- P-values for each marker and across markers using mMeta
- A p-value across markers using aMeta
- Optional forest plot per marker and across markers via mMeta.aMeta.plot

## How to apply

Install the mMeta R package from GitHub via devtools (install_github('hk1785/mMeta')), then supply a study-by-marker matrix of regression coefficients (est) and a matching matrix of standard errors (std.err); NAs must be matched between the two matrices and the number of studies must be ≥10. Call mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123) to fit both mMeta and aMeta under the Han and Eskin modification (heterogeneous variance does not exist under the null, which avoids the over-conservatism of the traditional random effects null), using the SJ heterogeneity estimator and group permutations. Set tau0=TRUE only when you deliberately want the traditional random effects meta-analysis that assumes heterogeneity exists under the null; comparing tau0=FALSE versus tau0=TRUE outputs (as for Figure 2, Figure 3, and Table S2) confirms the Han-Eskin modification yields smaller p-values. If the number of studies is <15, all possible permutations are implemented and n.perm and seed are ignored. Extract per-marker mMeta p-values and the aMeta p-value across markers, and optionally pass the fitted object to mMeta.aMeta.plot to draw forest plots for each marker and across markers.

## Related tools

- **mMeta** (R package providing mMeta.aMeta (multi-marker meta-analysis core function), mMeta.aMeta.plot (forest plots), and the HIV.Sum.Data example dataset) — https://github.com/hk1785/mMeta
- **R** (Runtime environment for mMeta and aMeta implementation (R >= 4.0.3; imports forestplot, MASS, vegan)) — https://github.com/hk1785/mMeta

## Examples

```
out <- mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123); out; mMeta.aMeta.plot(mMeta.aMeta.out=out)
```

## Evaluation signals

- The input matrices est and std.err have identical dimensions and matched NA locations, with ≥10 study rows and named marker columns
- Fitting with tau0=FALSE (Han-Eskin modification) produces p-values no larger than the tau0=TRUE traditional random effects fit on the same data, seed, and permutation settings (as reported for Figure 2, Figure 3, and Table S2)
- Output object contains pooled estimates, 95% CIs, per-marker and across-marker mMeta p-values, and a single aMeta p-value across markers
- With <15 studies, permutations are exhaustive and results are reproducible regardless of seed; with ≥15 studies, the same seed (e.g., 123) reproduces identical p-values
- mMeta.aMeta.plot on the fitted object renders a forest plot for each marker and across markers without error

## Limitations

- mMeta has only moderate power because it estimates marker-by-marker correlations non-parametrically via permutations
- aMeta is limited to hypothesis testing (no pooled estimate or confidence interval)
- Effect estimates and standard errors required by mMeta and aMeta are hard to obtain in practice; many studies report only graphs (box plots, bar/line charts with error bars) or stratified summaries (mean, median, IQR per group) rather than difference/association estimates and p-values
- There are no set criteria for selecting candidate markers; selection rules should be explicitly delineated, and the six α-diversity indices are not always available from prior studies
- No version number is stated for the mMeta R package and no changelog is found for the repository

## Evidence

- [other] Han-Eskin modification rationale (source card finding): [evidence span withheld — anchor status: not_found]
