---
name: simulation-based-power-and-type-i-error-estimation
description: Use when comparing statistical meta-analysis or testing methods' empirical
  power and type I error across scenarios, using simulated summary statistics rather
  than real data.
when_to_use_negative:
- The goal is estimation of a pooled effect (mMeta provides pooled estimates and 95%
  CIs, but aMeta is limited to hypothesis testing with no estimation facilities)
- Only one study is available — mMeta.aMeta requires at least 10 studies (rows in
  est/std.err)
- Raw microbiome sequences need processing (quality control, OTU picking) rather than
  meta-analysis of already-computed summary statistics
edam_operation: http://edamontology.org/operation_3578
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Implementation environment for the simulation, per-study regression fitting,
    and meta-analysis evaluation
  repo: https://github.com/hk1785/mMeta
- name: R package mMeta
  role: Provides mMeta.aMeta for running mMeta and aMeta with the SJ estimator and
    group permutations on simulated summary statistics
  repo: https://github.com/hk1785/mMeta
- name: vegan
  role: R package imported by mMeta used for computing α-diversity indices from the
    simulated count tables
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:c9061ec3a8b863a1ad8a3548af7fcc1d39e465d4e7bc453930519166108f5b1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - vegan
  edam_operation: http://edamontology.org/operation_3578
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# simulation-based-power-and-type-i-error-estimation

## Summary

In-silico evaluation of meta-analysis methods (mMeta, aMeta, and single-marker meta-analyses) by simulating microbiome count data from a Dirichlet-multinomial distribution and estimating empirical power at the 5% significance level, plus type I error under the null.

## When to use

Use when comparing statistical meta-analysis or testing methods' empirical power and type I error across scenarios, using simulated summary statistics rather than real data.

## When NOT to use

- The goal is estimation of a pooled effect (mMeta provides pooled estimates and 95% CIs, but aMeta is limited to hypothesis testing with no estimation facilities)
- Only one study is available — mMeta.aMeta requires at least 10 studies (rows in est/std.err)
- Raw microbiome sequences need processing (quality control, OTU picking) rather than meta-analysis of already-computed summary statistics

## Inputs

- Simulated per-study count tables of samples by species (Dirichlet-multinomial, 100 OTUs, K=10/20/30 studies, 50-100 samples per study, 1,000-5,000 total reads per sample)
- Phylogenetic tree for the 100 surrogate OTUs from the respiratory-tract microbiome data
- Per-study linear regression summary statistics: effect estimates β^j,k and standard errors σ^j,k for each of the six α-diversity indices (Richness, Shannon, Simpson, PD, PE, PQE)

## Outputs

- Empirical power estimates at the 5% significance level for mMeta, aMeta, and each of the six marker-specific meta-analyses under each association scenario (ζ)
- Type I error rates at the 5% significance level under the null (βk=0)
- Power comparison plot (analogous to Figure 1 of the paper)

## How to apply

Simulate count tables across samples and species per study (K=10, 20, 30) from the Dirichlet-multinomial distribution using 100 OTUs and a phylogenetic tree from respiratory-tract microbiome data, with per-study sample sizes drawn from a discrete uniform distribution on 50-100 and total reads from 1,000-5,000 in R using the mMeta package's simulation utilities (vegan for diversity computation). Generate Gaussian host-phenotype responses via y = 0.5*scale(x1) + 0.5*scale(x2) + βk*scale(sum of counts in ζ) + ε, with βk=1 for power estimation under four association-scenario sets ζ (10 rarest species, 10 random species, 10 most common species, and a random PAM-clustered phylogenetic cluster); set βk=0 for type I error estimation. Compute the six α-diversity indices (Species richness, Shannon, Simpson, PD, PE, PQE) per sample with the phylogenetic tree, fit per-study linear regressions adjusting for x1 and x2 by least squares to obtain β^j,k and σ^j,k, then run mMeta.aMeta with the SJ heterogeneity-variance estimator, the Han and Eskin modified null (tau0=FALSE, no heterogeneity under H0), n.perm=5000 group permutations, and the single-marker meta-analyses per index. Estimate empirical power (or type I error under the null) at the 5% significance level for each marker-specific meta-analysis and for mMeta and aMeta under each scenario, and render the power comparison plot.

## Related tools

- **R** (Implementation environment for the simulation, per-study regression fitting, and meta-analysis evaluation) — https://github.com/hk1785/mMeta
- **R package mMeta** (Provides mMeta.aMeta for running mMeta and aMeta with the SJ estimator and group permutations on simulated summary statistics) — https://github.com/hk1785/mMeta
- **vegan** (R package imported by mMeta used for computing α-diversity indices from the simulated count tables) — https://github.com/hk1785/mMeta

## Examples

```
library(mMeta); out <- mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Reproduced figure should show mMeta retaining medium power across all four scenarios (ζ) while aMeta closely approaches the highest power among the six single-marker meta-analyses, matching Figure 1 of the paper
- Under the null (βk=0), type I error rates should be at most 5% per Table 3
- Group permutation method should be more powerful than the asymptotic normality method (Figure 1A,B < Figure 1C,D)
- mMeta.aMeta must be called with ≥10 studies (rows in est and std.err); aMeta's p-value across markers should be attainable for all four scenario sets
- The Han and Eskin modified null setting (tau0=FALSE) should be used, distinct from the traditional random-effects null (tau0=TRUE)

## Limitations

- aMeta is limited to hypothesis testing with no estimation facilities (no pooled effect estimates or confidence intervals)
- mMeta's power is only moderate because marker-by-marker correlations are estimated non-parametrically via permutations
- mMeta.aMeta requires at least 10 studies; with fewer than 15 studies all possible permutations are implemented and n.perm/seed are ignored
- No set criteria exist for selecting candidate markers for mMeta and aMeta; which selection rules are most reasonable is debatable

## Evidence
