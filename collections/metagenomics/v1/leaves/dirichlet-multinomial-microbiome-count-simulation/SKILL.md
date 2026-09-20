---
name: dirichlet-multinomial-microbiome-count-simulation
description: Use when you need to evaluate or benchmark microbiome meta-analysis methods
  (mMeta, aMeta, marker-specific meta-analysis) under known association scenarios
  using simulated multi-study count data and α-diversity indices.
when_to_use_negative:
- When the goal is pooled effect estimation via aMeta — aMeta is limited to hypothesis
  testing with no estimation facilities
- When fewer than 10 studies are available — mMeta.aMeta requires the number of studies
  to be at least 10
- When raw individual-level microbiome data must be analyzed directly rather than
  summary statistics of effect estimates and standard errors
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mMeta
  role: R package providing the simulation utilities, Dirichlet-multinomial count-table
    generation, and the mMeta.aMeta function for meta-analysis of simulated summary
    statistics
  repo: https://github.com/hk1785/mMeta
- name: vegan
  role: R package imported by mMeta used to compute the six α-diversity indices from
    the simulated count tables
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Environment in which the simulation, regression fitting, and meta-analysis
    are executed (R >= 4.0.3 required)
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:8ce1812e918504e2a66fccdc4e6b8d88b4d5a67e8db46c5db0e76e6bcc7c248f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  - vegan
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# dirichlet-multinomial-microbiome-count-simulation

## Summary

In-silico evaluation scheme for microbiome meta-analysis methods: generate multi-study OTU count tables from the Dirichlet-multinomial distribution using the 100 respiratory-tract OTUs and phylogenetic tree, then derive α-diversity summary statistics to estimate empirical power and type I error of mMeta and aMeta against marker-specific meta-analyses.

## When to use

Use when you need to evaluate or benchmark microbiome meta-analysis methods (mMeta, aMeta, marker-specific meta-analysis) under known association scenarios using simulated multi-study count data and α-diversity indices.

## When NOT to use

- When the goal is pooled effect estimation via aMeta — aMeta is limited to hypothesis testing with no estimation facilities
- When fewer than 10 studies are available — mMeta.aMeta requires the number of studies to be at least 10
- When raw individual-level microbiome data must be analyzed directly rather than summary statistics of effect estimates and standard errors

## Inputs

- 100 common OTUs and their phylogenetic tree from the respiratory-tract microbiome data (surrogates for 100 microbial species)
- Simulation parameters: number of studies K (10, 20, or 30), per-study sample sizes (discrete uniform 50–100), total reads per sample (1,000–5,000)
- Association scenario definition ζ and effect size βk (0 for null, 1 for power estimation)
- Covariates x1 (Bernoulli) and x2 (N(0,1))
- Study-specific effect estimates and standard errors in est/std.err data.frames for the meta-analysis step

## Outputs

- Simulated Dirichlet-multinomial OTU count tables per study per scenario
- Six α-diversity indices per sample (Species richness, Shannon, Simpson, PD, PE, PQE)
- Per-study regression coefficients and standard errors for each α-diversity index
- Empirical power and type I error estimates at the 5% significance level for mMeta, aMeta, and marker-specific meta-analyses
- Power comparison plot analogous to Figure 1 of the paper

## How to apply

Install the mMeta R package from GitHub (install_github('hk1785/mMeta', force=T)) and use its simulation utilities with vegan for diversity computation. Simulate count tables across samples and species per study (K=10, 20, 30 studies) from the Dirichlet-multinomial distribution using the 100 common OTUs and their phylogenetic tree from the respiratory-tract microbiome data, drawing per-study sample sizes from a discrete uniform on 50–100 and total reads from 1,000–5,000. Generate Gaussian host phenotypes via y = 0.5*scale(x1) + 0.5*scale(x2) + βk*scale(sum of counts in ζ) + ε, with βk=0 for type I error and βk=1 under four association scenarios ζ (S1: 10 rarest species; S2: 10 random species; S3: 10 most common species; S4: a random PAM-clustered phylogenetic cluster). Compute the six α-diversity indices (Species richness, Shannon, Simpson, PD, PE, PQE) per sample, then fit per-study least-squares linear regressions adjusting for covariates x1 (Bernoulli) and x2 (N(0,1)) to obtain study-specific effect estimates β^j,k and standard errors σ^j,k. Meta-analyze the summary statistics with mMeta.aMeta using the SJ heterogeneity-variance estimator with the Han and Eskin modified null (tau0=FALSE) and n.perm=5000 group permutations, alongside per-index single-marker meta-analyses. Estimate empirical power and type I error at the 5% significance level and render a power comparison plot analogous to Figure 1.

## Related tools

- **mMeta** (R package providing the simulation utilities, Dirichlet-multinomial count-table generation, and the mMeta.aMeta function for meta-analysis of simulated summary statistics) — https://github.com/hk1785/mMeta
- **vegan** (R package imported by mMeta used to compute the six α-diversity indices from the simulated count tables) — https://github.com/hk1785/mMeta
- **R** (Environment in which the simulation, regression fitting, and meta-analysis are executed (R >= 4.0.3 required)) — https://github.com/hk1785/mMeta

## Examples

```
out <- mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Under the null (βk=0), empirical type I error rates are well-controlled at at most 5% for all surveyed methods
- The reproduced figure should show mMeta retaining medium power across all scenarios and aMeta approaching the highest power among the six single-marker meta-analyses, matching Figure 1 of the paper
- SJ-based power estimates exceed DL-based ones (Figure 1A,C,E < Figure 1B,D,F)
- Han-Eskin modified analyses (tau0=FALSE) produce smaller p-values than traditional random effects analyses (Figure 3B vs Figure 3A)
- The group permutation method yields higher power than the method based on asymptotic normality

## Limitations

- mMeta's power is only moderate because it estimates marker-by-marker correlations non-parametrically via permutations
- aMeta closely approaches the highest power among marker-specific meta-analyses yet is limited to hypothesis testing with no estimation facilities
- If the number of studies < 15, all possible permutations are implemented and n.perm and seed are ignored
- Effect estimates and standard errors required by mMeta and aMeta are hard to obtain in practice, and the six α-diversity indices are not always available from prior studies
- There are no set criteria for selecting candidate markers; the selection rules used should be documented for future readers

## Evidence
