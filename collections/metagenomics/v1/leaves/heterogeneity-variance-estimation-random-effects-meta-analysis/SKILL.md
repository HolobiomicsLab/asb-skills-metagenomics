---
name: heterogeneity-variance-estimation-random-effects-meta-analysis
description: Decision support for choosing and implementing the heterogeneity variance
  estimator (SJ vs DL) when meta-analyzing study-specific effect estimates and standard
  errors across multiple related markers, e.g. alpha-diversity indices.
when_to_use_negative:
- The number of studies (rows of est/std.err) is fewer than 10 — mMeta.aMeta requires
  at least 10 studies
- Only raw sequence data or OTU tables are available with no study-specific effect
  estimates and standard errors, since mMeta and aMeta require only effect estimates
  and their standard errors, which must already be computed (e.g. via least squares
  regression adjusting for x1 and x2)
- Fixed-effects meta-analysis is appropriate (no between-study heterogeneity expected),
  as this skill exists specifically to estimate the random effects heterogeneity variance
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: mMeta
  role: R package implementing mMeta and aMeta random effects meta-analysis with the
    SJ heterogeneity variance estimator via mMeta.aMeta(est, std.err, tau0=FALSE,
    n.perm=5000, seed=123)
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Environment in which the DL estimator variant of the single-marker and multi-marker
    meta-analysis (Eqs. 2-6) is implemented and simulations are run
- name: R package mMeta
  role: Installable via devtools install_github('hk1785/mMeta', force=T); provides
    HIV.Sum.Data summary statistics and the mMeta.aMeta/mMeta.aMeta.plot functions
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:6379bb5efd09691ee1696a482d63ed8b22f3dec9d61442c8a50a77100c465250
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# heterogeneity-variance-estimation-random-effects-meta-analysis

## Summary

Method for estimating the between-study (heterogeneity) variance tau^2 in random effects meta-analysis by choosing between the Sidik-Jonkman (SJ/robust sandwich) estimator and the DerSimonian-Laird (DL) method-of-moments estimator. The Koh & Zhao (2021) mMeta/aMeta framework adopts SJ because it refines an initial heterogeneity variance estimate, yielding a robust estimate and higher power.

## When to use

Decision support for choosing and implementing the heterogeneity variance estimator (SJ vs DL) when meta-analyzing study-specific effect estimates and standard errors across multiple related markers, e.g. alpha-diversity indices.

## When NOT to use

- The number of studies (rows of est/std.err) is fewer than 10 — mMeta.aMeta requires at least 10 studies
- Only raw sequence data or OTU tables are available with no study-specific effect estimates and standard errors, since mMeta and aMeta require only effect estimates and their standard errors, which must already be computed (e.g. via least squares regression adjusting for x1 and x2)
- Fixed-effects meta-analysis is appropriate (no between-study heterogeneity expected), as this skill exists specifically to estimate the random effects heterogeneity variance

## Inputs

- Study-specific effect estimates matrix (est) with rows as studies and columns as markers (>= 10 studies; NAs allowed if matched between est and std.err)
- Matching study-specific standard errors matrix (std.err)
- Six alpha-diversity indices (Species richness, Shannon, Simpson, PD, PE, PQE) per sample from rarefied OTU tables
- Summary statistics dataset HIV.Sum.Data (15 HIV studies x 6 alpha-diversity indices)
- Simulated Dirichlet-multinomial OTU count tables for K=10, 20, 30 studies with covariates x1 (Bernoulli) and x2 (N(0,1))

## Outputs

- Marker-specific and multi-marker (mMeta/aMeta) meta-analysis p-values under SJ and DL tau^2 estimators
- Empirical type I error rates and power estimates at the 5% level per scenario (sj_dl_comparison.csv)
- Power comparison figure analogous to Figure 1 (sj_dl_comparison.png)

## How to apply

Given a summary data matrix of study-specific regression coefficients (est) and standard errors (std.err) — e.g. 15 HIV studies x 6 alpha-diversity indices (Richness/Observed, Shannon, Simpson, PD, PE, PQE) from HIV.Sum.Data — estimate tau^2 for each marker's random effects meta-analysis. The paper's mMeta and aMeta use the Sidik-Jonkman (SJ) estimator (robust/sandwich variance) because it gives a robust estimate based on a refinement using an initial heterogeneity variance estimate; the comparison task swaps in the DerSimonian-Laird (DL) method-of-moments estimator (Eqs. 2-6) as the alternative. Run both variants through the group permutation method (n.perm=5000, seed=123), under both tau0=FALSE (Han-Eskin modification: no heterogeneity under H0) and tau0=TRUE (traditional random effects: heterogeneity under H0), for both single-marker and multi-marker (mMeta/aMeta) tests. Compare empirical power at the 5% level via simulations (K=10, 20, 30 Dirichlet-multinomial studies on 100 respiratory-tract OTUs, sample sizes 50-100, reads 1000-5000, beta_k=1 under S1-S4 scenarios), then verify SJ outperforms DL while both control type I error at <= 5%.

## Related tools

- **mMeta** (R package implementing mMeta and aMeta random effects meta-analysis with the SJ heterogeneity variance estimator via mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123)) — https://github.com/hk1785/mMeta
- **R** (Environment in which the DL estimator variant of the single-marker and multi-marker meta-analysis (Eqs. 2-6) is implemented and simulations are run)
- **R package mMeta** (Installable via devtools install_github('hk1785/mMeta', force=T); provides HIV.Sum.Data summary statistics and the mMeta.aMeta/mMeta.aMeta.plot functions) — https://github.com/hk1785/mMeta

## Examples

```
library(mMeta); data(HIV.Sum.Data); out <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Empirical type I error rates are well-controlled at <= 5% for all surveyed methods (both SJ and DL)
- SJ-based power estimates exceed DL-based ones (Figure 1A,C,E < Figure 1B,D,F)
- Han-Eskin modified analyses (tau0=FALSE) produce smaller p-values than traditional random effects analyses (tau0=TRUE) (Figure 3B vs Figure 3A, Table S2)
- Marker-specific and multi-marker p-values are reproducible with seed=123 and n.perm=5000
- Aggregate results table sj_dl_comparison.csv contains power and type I error estimates for all simulation scenarios (S1-S4) and real-data analyses

## Limitations

- mMeta.aMeta requires at least 10 studies; if the number of studies < 15, all possible permutations are implemented and n.perm/seed are ignored
- Effect estimates and their standard errors required by mMeta and aMeta are hard to obtain in practice; many studies report only graphs or stratified summaries (mean, median, IQR per group)
- NAs in est and std.err are allowed but their locations must be matched between the two matrices
- The DL comparison requires re-implementing the method-of-moments estimator in R, as the mMeta package implements only the SJ estimator

## Evidence

- [other] mMeta and aMeta estimate the heterogeneity variance using the Sidik and Jonkman (SJ) estimator (also called the robust or sandwich variance estimator): [evidence span withheld — anchor status: not_found]
- [methods] data(HIV.Sum.Data) # Summary data for 15 HIV studies and 6 alpha-diversity indices: [evidence span withheld — anchor status: not_found]
