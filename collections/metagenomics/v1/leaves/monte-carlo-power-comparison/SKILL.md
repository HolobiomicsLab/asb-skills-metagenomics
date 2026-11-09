---
name: monte-carlo-power-comparison
description: Use when choosing a between-study heterogeneity variance estimator (SJ
  vs DL) for random effects meta-analysis across multiple related markers, or when
  validating empirical power/type I error of marker-set meta-analysis methods via
  simulations.
when_to_use_negative:
- Input is raw 16S rRNA sequence data or an unprocessed OTU table — this skill requires
  study-level summary statistics (effect estimates and their standard errors), which
  mMeta and aMeta alone cannot extract from graphs or stratified means/medians
- 'Fewer than 10 studies: mMeta.aMeta requires the number of studies >= 10 rows, and
  with < 15 studies all permutations are exhaustively enumerated (n.perm and seed
  are ignored), so permutation-count-based power comparisons do not apply as designed'
- Fixed-effects meta-analysis problems without between-study heterogeneity — the SJ
  and DL estimators exist to estimate heterogeneity variance and the comparison is
  meaningless under homogeneity
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mMeta
  role: R package providing mMeta.aMeta/mMeta.aMeta.plot functions that run the group-permutation
    random effects meta-analysis with the SJ heterogeneity variance estimator used
    as the baseline in the comparison
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Environment in which the simulations (Dirichlet-multinomial generation, six
    alpha-diversity index computation, least squares regression) and the DL estimator
    re-implementation of Eqs. 2-6 are coded
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:49be1a484a3348b613fe13cb592c251646f0450644c5da37f2cf97f3f538ff47
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# monte-carlo-power-comparison

## Summary

Monte Carlo simulation and real-data benchmark that compares the Sidik-Jonkman (SJ) heterogeneity variance estimator against the DerSimonian-Laird (DL) method-of-moments estimator within the mMeta/aMeta random effects meta-analysis framework, demonstrating that SJ yields higher power while keeping type I error <= 5%.

## When to use

Use when choosing a between-study heterogeneity variance estimator (SJ vs DL) for random effects meta-analysis across multiple related markers, or when validating empirical power/type I error of marker-set meta-analysis methods via simulations.

## When NOT to use

- Input is raw 16S rRNA sequence data or an unprocessed OTU table — this skill requires study-level summary statistics (effect estimates and their standard errors), which mMeta and aMeta alone cannot extract from graphs or stratified means/medians
- Fewer than 10 studies: mMeta.aMeta requires the number of studies >= 10 rows, and with < 15 studies all permutations are exhaustively enumerated (n.perm and seed are ignored), so permutation-count-based power comparisons do not apply as designed
- Fixed-effects meta-analysis problems without between-study heterogeneity — the SJ and DL estimators exist to estimate heterogeneity variance and the comparison is meaningless under homogeneity

## Inputs

- Study-by-marker matrix of effect estimates (est; rows = studies, columns = markers, NAs allowed if matched between est and std.err; >= 10 studies)
- Study-by-marker matrix of standard errors (std.err)
- HIV.Sum.Data summary dataset (15 HIV studies x 6 alpha-diversity indices) from the mMeta package
- Simulated Dirichlet-multinomial OTU count tables (K=10/20/30 studies, 100 respiratory-tract OTUs, sample sizes 50-100, total reads 1,000-5,000)
- Phylogenetic tree of the 100 surrogate OTUs (for PD, PE, PQE indices)

## Outputs

- Marker-specific and multi-marker p-values from mMeta and aMeta under SJ and DL estimators (tau0=FALSE and tau0=TRUE)
- Empirical type I error rates at the 5% level for each estimator/scenario
- Empirical power estimates for scenarios S1-S4 with beta_k=1
- sj_dl_comparison.csv (aggregated comparison table)
- sj_dl_comparison.png (power comparison figure analogous to Figure 1)

## How to apply

Install the mMeta R package from GitHub (devtools::install_github('hk1785/mMeta', force=T)) and run mMeta.aMeta on the HIV.Sum.Data summary statistics (15 HIV studies x 6 alpha-diversity indices: Richness/Observed, Shannon, Simpson, PD, PE, PQE) with n.perm=5000 and seed=123 under both tau0=FALSE (Han-Eskin modification) and tau0=TRUE (traditional random effects). Implement the DL variant by replacing the SJ (robust/sandwich) step with the method-of-moments estimator in the single- and multi-marker meta-analysis equations (Eqs. 2-6), then re-run the real-data analyses. For simulations, generate Dirichlet-multinomial count tables for K=10, 20, 30 studies using the 100 respiratory-tract OTUs and their phylogenetic tree, with sample sizes drawn from discrete uniform 50-100 and total reads from 1,000-5,000; set beta_k=0 for type I error and beta_k=1 under scenarios S1 (10 rarest species), S2 (10 random species), S3 (10 most common species) and S4 (a PAM phylogenetic cluster). Compute the six alpha-diversity indices per sample, fit least squares linear regression adjusting for x1 (Bernoulli) and x2 (N(0,1)), and meta-analyze with both SJ and DL via the group permutation method at the 5% level. Aggregate results into sj_dl_comparison.csv and draw a Figure 1-style power comparison (sj_dl_comparison.png).

## Related tools

- **mMeta** (R package providing mMeta.aMeta/mMeta.aMeta.plot functions that run the group-permutation random effects meta-analysis with the SJ heterogeneity variance estimator used as the baseline in the comparison) — https://github.com/hk1785/mMeta
- **R** (Environment in which the simulations (Dirichlet-multinomial generation, six alpha-diversity index computation, least squares regression) and the DL estimator re-implementation of Eqs. 2-6 are coded) — https://github.com/hk1785/mMeta

## Examples

```
library(devtools); install_github('hk1785/mMeta', force=T); data(HIV.Sum.Data); out2 <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Empirical type I error rates are well-controlled at <= 5% for all surveyed methods when beta_k=0
- SJ-based power estimates exceed DL-based ones across S1-S4 (Figure 1A,C,E panels < Figure 1B,D,F panels)
- Han-Eskin modified analyses (tau0=FALSE) produce smaller p-values than the traditional random effects analyses (tau0=TRUE) on the real HIV data (Figure 3B vs Figure 3A, Table S2)
- est and std.err matrices have NA locations matched between them and at least 10 studies (rows)
- Group permutation method yields greater power than the asymptotic normality method in the same simulated data

## Limitations

- The SJ estimator is the only heterogeneity variance estimator implemented in the released mMeta package (version 1.0, 2021-2-11); DL comparisons require custom re-implementation of Eqs. 2-6
- If the number of studies < 15, all possible permutations are implemented and n.perm/seed are ignored, limiting Monte Carlo control over permutation sampling
- Effect estimates and their standard errors required by mMeta and aMeta are hard to obtain in practice — many studies report only graphs or stratified summary statistics rather than association estimates with SEs
- No set criteria exist for selecting candidate markers for mMeta and aMeta, so simulation-based power results depend on the marker selection rules chosen

## Evidence

- [other] SJ estimator rationale: [evidence span withheld — anchor status: not_found]
- [intro] Summary data requirement: [evidence span withheld — anchor status: not_found]
