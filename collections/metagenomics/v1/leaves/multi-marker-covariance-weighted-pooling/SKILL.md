---
name: multi-marker-covariance-weighted-pooling
description: 'Use when combining ≥10 studies across multiple related markers (e.g.,
  six α-diversity indices: Richness, Shannon, Simpson, PD, PE, PQE) from summary statistics
  alone, with no priori marker-by-marker correlation estimates and possible missing
  data.'
when_to_use_negative:
- You have raw sequence data (e.g., FASTQ/16S reads) rather than summary statistics
  — mMeta/aMeta require only effect estimates and standard errors, not raw OTU tables
- You have < 10 studies or fewer rows in est/std.err than the minimum required by
  the package
- You need flexible handling of many different types of summary data (e.g., only graphs,
  stratified means/medians/IQRs per group) — the authors note no fully flexible method
  was achieved in this paper
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: mMeta
  role: R package implementing mMeta.aMeta (multi-marker meta-analysis and adaptive
    multi-marker meta-analysis) and mMeta.aMeta.plot on summary statistics
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment (R ≥ 4.0.3) required by the mMeta package
  repo: https://github.com/hk1785/mMeta
- name: devtools
  role: Used to install the mMeta package from GitHub via install_github("hk1785/mMeta")
- name: forestplot
  role: Imported dependency supporting forest plot rendering in mMeta.aMeta.plot
- name: MASS
  role: Imported dependency of the mMeta package
- name: vegan
  role: Imported dependency of the mMeta package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:6ddac42badbd4c8068d6fce485a695170bfbb3a2db318c50962b5dcc44f4afb0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - MASS
  - R
  - R package mMeta
  - devtools
  - forestplot
  - mMeta
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# multi-marker-covariance-weighted-pooling

## Summary

Applies the mMeta and aMeta methods (R package mMeta) to combine multiple studies across multiple related markers using only summary statistics (effect estimates and standard errors), without prior marker-by-marker correlations. mMeta yields a covariance-weighted pooled estimate with its SE across all studies and markers, while aMeta delivers a more powerful across-marker hypothesis test via a minimum-p-value approach.

## When to use

Use when combining ≥10 studies across multiple related markers (e.g., six α-diversity indices: Richness, Shannon, Simpson, PD, PE, PQE) from summary statistics alone, with no priori marker-by-marker correlation estimates and possible missing data.

## When NOT to use

- You have raw sequence data (e.g., FASTQ/16S reads) rather than summary statistics — mMeta/aMeta require only effect estimates and standard errors, not raw OTU tables
- You have < 10 studies or fewer rows in est/std.err than the minimum required by the package
- You need flexible handling of many different types of summary data (e.g., only graphs, stratified means/medians/IQRs per group) — the authors note no fully flexible method was achieved in this paper

## Inputs

- est: data.frame or matrix of study-specific regression coefficients (rows = studies, columns = markers; ≥10 studies; NAs allowed but must match std.err locations), e.g., HIV.Sum.Data$est (15 HIV studies × 6 α-diversity indices)
- std.err: data.frame or matrix of study-specific standard errors matched to est, e.g., HIV.Sum.Data$std.err
- Optional parameters: tau0 (heterogeneous variance indicator), n.perm (default 5000), seed

## Outputs

- Per-marker pooled estimates, 95% confidence intervals, and p-values (mMeta)
- Across-marker pooled estimate, 95% CI, and p-value (mMeta)
- Across-marker p-value (aMeta, minimum-p-value test)
- Optional forest plot per marker and across markers via mMeta.aMeta.plot
- Results table, e.g., mmeta_ameta_hiv_results.csv

## How to apply

Install the mMeta R package from GitHub with devtools (install_github("hk1785/mMeta")) and load forestplot, MASS, and vegan dependencies. Assemble a data.frame/matrix of study-specific regression coefficients ('est') and matched standard errors ('std.err'), rows = studies (row names = study names), columns = markers (column names = marker names), with ≥10 studies and NAs matched between the two matrices. Call mMeta.aMeta(est, std.err, seed=123) which runs single-marker random effects meta-analysis with the SJ heterogeneity estimator plus sign-flipping group permutation (n.perm=5000; if studies < 15, all possible permutations are used and n.perm/seed are ignored), then covariance-aware mMeta weighted pooling across markers and the aMeta minimum-p-value test; set tau0=FALSE for Han–Eskin's modified random effects if heterogeneity-free nulls are desired. Extract per-marker pooled estimates, 95% CIs and p-values, the mMeta across-marker pooled estimate/CI/p-value, and the aMeta across-marker p-value, then confirm the aMeta p-value is smaller (more powerful) than the mMeta p-value before writing results (e.g., mmeta_ameta_hiv_results.csv).

## Related tools

- **mMeta** (R package implementing mMeta.aMeta (multi-marker meta-analysis and adaptive multi-marker meta-analysis) and mMeta.aMeta.plot on summary statistics) — https://github.com/hk1785/mMeta
- **R** (Runtime environment (R ≥ 4.0.3) required by the mMeta package) — https://github.com/hk1785/mMeta
- **devtools** (Used to install the mMeta package from GitHub via install_github("hk1785/mMeta"))
- **forestplot** (Imported dependency supporting forest plot rendering in mMeta.aMeta.plot)
- **MASS** (Imported dependency of the mMeta package)
- **vegan** (Imported dependency of the mMeta package)

## Examples

```
library(mMeta); data(HIV.Sum.Data); est <- HIV.Sum.Data$est; std.err <- HIV.Sum.Data$std.err; out2 <- mMeta.aMeta(est, std.err, seed=123); out2
```

## Evaluation signals

- Output includes per-marker pooled estimates, 95% CIs, and p-values for all six α-diversity markers plus an across-marker mMeta estimate and an across-marker aMeta p-value
- The aMeta across-marker p-value is smaller (more powerful) than the mMeta across-marker p-value
- Computed p-values match those reported in Figure 3 and Table S2 of the paper when run on HIV.Sum.Data with seed=123
- The est and std.err matrices have matched NA locations and ≥ 10 studies (rows); if studies < 15, all possible permutations are exhausted and n.perm/seed are ignored
- With 15 HIV studies and six indices (Richness, Shannon, Simpson, PD, PE, PQE), the run reproduces the 15×6 'est' and 'std.err' structures and writes a complete results table (mmeta_ameta_hiv_results.csv)

## Limitations

- aMeta is limited to hypothesis testing only — it provides an across-marker p-value but no pooled estimate, whereas mMeta provides the pooled estimate and SE
- mMeta's power for hypothesis testing is only moderate, though it approaches marker-specific meta-analyses; aMeta closely approaches the highest power among marker-specific meta-analyses
- Effect estimates and their standard errors required by mMeta and aMeta are hard to obtain in practice — many studies report only graphs (box plots, bar/line charts with error bars) with no numbers, or stratified effect estimates (mean, median, IQR per group)
- No set criteria exist for selecting candidate markers for mMeta and aMeta; the authors suggest delineating selection rules so future readers can evaluate reasonableness
- The six α-diversity indices (Richness, Shannon, Simpson, PD, PE, PQE) are not always available from prior studies while other indices may be available
- No version number stated for the mMeta R package and no changelog found for the repository

## Evidence

- [methods] data(HIV.Sum.Data) # Summary data for 15 HIV studies and 6 alpha-diversity indices: [evidence span withheld — anchor status: not_found]
