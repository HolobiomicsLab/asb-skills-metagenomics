---
name: statistical-power-comparison-across-meta-analysis-methods
description: Use when meta-analyzing summary statistics (effect estimates and standard
  errors) across multiple related markers and you need to decide between or verify
  that the group permutation p-value method outperforms asymptotic normality.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/OTU tables) rather than summary statistics —
  mMeta/aMeta require only effect estimates and their standard errors, which are hard
  to obtain from studies reporting only graphs or stratified summary values
- Fewer than 10 studies are available — the mMeta.aMeta function requires the number
  of studies (rows) to be >= 10
- Marker-by-marker correlation estimates (not just p-value/power comparison) are the
  goal — use mMeta's non-parametric permutation correlation estimation instead, since
  aMeta is limited to hypothesis testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: mMeta
  role: R package providing mMeta.aMeta to run both p-value calculation methods (group
    permutation and asymptotic normality) on study-specific effect estimates and standard
    errors
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment; package depends on R (>= 4.0.3) and imports forestplot,
    MASS, vegan
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:78d06d898c34f7fd87392db382504138cb4eda9ac4b866053fefb92732e61360
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# statistical-power-comparison-across-meta-analysis-methods

## Summary

Benchmarks the p-value calculation methods underlying mMeta/aMeta — the semi-parametric group permutation method (Follmann-Proschan) versus the parametric method based on asymptotic normality (mu_hat/sigma_mu ~ N(0,1)) — by comparing p-values and power at the 5% significance level across markers.

## When to use

Use when meta-analyzing summary statistics (effect estimates and standard errors) across multiple related markers and you need to decide between or verify that the group permutation p-value method outperforms asymptotic normality.

## When NOT to use

- Input is raw sequencing data (FASTQ/OTU tables) rather than summary statistics — mMeta/aMeta require only effect estimates and their standard errors, which are hard to obtain from studies reporting only graphs or stratified summary values
- Fewer than 10 studies are available — the mMeta.aMeta function requires the number of studies (rows) to be >= 10
- Marker-by-marker correlation estimates (not just p-value/power comparison) are the goal — use mMeta's non-parametric permutation correlation estimation instead, since aMeta is limited to hypothesis testing

## Inputs

- est: data.frame/matrix of study-specific effect estimates (rows = studies, columns = markers, NAs matched between est and std.err, >= 10 studies)
- std.err: data.frame/matrix of study-specific standard errors (row/column names matching est)
- HIV.Sum.Data: summary data for 15 HIV studies and 6 alpha-diversity indices (Richness, Shannon, Simpson, PD, PE, PQE)
- Sum.Data: example packaged summary data (est and std.err)

## Outputs

- Pooled estimates, 95% confidence intervals, and p-values for each marker and across markers using mMeta
- A p-value across markers using aMeta (minimum p-value statistic with permutation null)
- Side-by-side comparison table of p-values and power estimates from the group permutation vs asymptotic normality methods (e.g., pvalue_method_comparison.csv)

## How to apply

Install the mMeta R package from GitHub with devtools (install_github('hk1785/mMeta', force=T)) and load the packaged summary datasets (Sum.Data or HIV.Sum.Data, which holds effect estimates and standard errors for 15 HIV studies across 6 alpha-diversity indices: Richness, Shannon, Simpson, PD, PE, PQE). Run mMeta.aMeta with the Follmann-Proschan group permutation method (n.perm=5000, seed=123, tau0=FALSE for the Han-Eskin no-heterogeneity-under-H0 modification) and with the parametric asymptotic-normality method for each marker and across markers. Optionally replicate the simulation design (K=10, 20 or 30 studies; Dirichlet-multinomial count tables from 100 respiratory-tract OTUs; scenarios S1 rare, S2 random, S3 common, S4 PAM phylogenetic cluster) to estimate power at the 5% significance level. Tabulate p-values and power estimates side by side and confirm the ordering reported in Figure 1 and Table S2: the group permutation method yields smaller p-values and higher power than the asymptotic normality method. Note that when the number of studies is < 15, all possible permutations are implemented and n.perm/seed are ignored.

## Related tools

- **mMeta** (R package providing mMeta.aMeta to run both p-value calculation methods (group permutation and asymptotic normality) on study-specific effect estimates and standard errors) — https://github.com/hk1785/mMeta
- **R** (Runtime environment; package depends on R (>= 4.0.3) and imports forestplot, MASS, vegan) — https://github.com/hk1785/mMeta

## Examples

```
library(mMeta); data(HIV.Sum.Data); out2 <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Group permutation p-values are smaller than (or equal to) asymptotic-normality p-values for each marker and across markers, matching the Figure 1A,B < Figure 1C,D ordering
- Estimated power at the 5% significance level is higher under the group permutation method than the asymptotic normality method in the simulation scenarios (S1 rare, S2 random, S3 common, S4 PAM), consistent with Table S2
- Repeated runs with the same seed (e.g., seed=123) and n.perm=5000 return identical outcomes; with < 15 studies, all possible permutations are enumerated and seed/n.perm are ignored
- NA locations in est and std.err are matched, and the number of studies is >= 10 for the call to succeed
- The comparison table (e.g., pvalue_method_comparison.csv) contains p-values for every marker and the across-marker aMeta statistic under both methods

## Limitations

- aMeta is limited to hypothesis testing (no pooled estimate across markers); mMeta has only moderate power despite estimating marker-by-marker correlations non-parametrically via permutations
- Effect estimates and standard errors are hard to obtain in practice since many studies report only graphs or stratified summary values (means, medians, IQRs per group)
- No set criteria exist for selecting candidate markers, and the six alpha-diversity indices are not always available from prior studies; no version number or changelog is documented for the mMeta R package

## Evidence
