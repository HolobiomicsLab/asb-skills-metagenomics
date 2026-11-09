---
name: statistical-power-estimation-by-monte-carlo
description: Use when you must empirically compare detection power of two differential-abundance
  methods for zero-inflated relative abundance data across graded effect sizes, e.g.
  replicating the paper's Fig 1b simulation (mu1=0.5 vs mu2=0.4/0.3/0.2).
when_to_use_negative:
- Input is real observed relative abundance data to be tested for differential abundance
  rather than synthetic data for power estimation — use taxa.compare directly instead
- The competing methods model counts/absolute abundances (e.g. negative-binomial frameworks)
  rather than zero-inflated beta relative abundances with arcsin square-root transformation
- Analytical power formulas are available for the chosen model — Monte Carlo simulation
  is unnecessary overhead
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3307
tools:
- name: metamicrobiomeR
  role: Provides the GAMLSS-BEZI differential abundance testing arm of the power simulation
    via its gamlss-based comparison workflow (taxa.compare with propmed.rel='gamlss')
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Zero-inflated beta distribution used both to generate the synthetic simulation
    data (mu, alpha, phi parameters) and to fit the more powerful test arm
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: MaAsLin
  role: 'Source of the comparison-method rationale: LMAS applies arcsin square-root
    transformation of relative abundance data before linear modeling'
- name: R
  role: Implementation language for the simulation loop, metamicrobiomeR analyses,
    and ggplot2 power-curve rendering (R 3.4.2 or higher required)
  repo: https://github.com/nhanhocu/metamicrobiomeR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:8be8f38772580d32d2078bb6baa8ed692f178dfba37d13b8f8bb88063d1faa59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - MaAsLin
  - R
  - metamicrobiomeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Statistical power estimation by Monte Carlo simulation

## Summary

Empirical power estimation for microbiome differential-abundance tests by simulating zero-inflated beta (BEZI) relative-abundance data across effect sizes and comparing GAMLSS-BEZI (metamicrobiomeR) against the MaAsLin-style arcsin square-root transformed linear model (LMAS) over many replicates.

## When to use

Use when you must empirically compare detection power of two differential-abundance methods for zero-inflated relative abundance data across graded effect sizes, e.g. replicating the paper's Fig 1b simulation (mu1=0.5 vs mu2=0.4/0.3/0.2).

## When NOT to use

- Input is real observed relative abundance data to be tested for differential abundance rather than synthetic data for power estimation — use taxa.compare directly instead
- The competing methods model counts/absolute abundances (e.g. negative-binomial frameworks) rather than zero-inflated beta relative abundances with arcsin square-root transformation
- Analytical power formulas are available for the chosen model — Monte Carlo simulation is unnecessary overhead

## Inputs

- Simulation design parameters: mu1=0.5, mu2 in {0.4, 0.3, 0.2}, zero-inflation alpha=0.5, precision phi=5, n=100 per group
- Number of Monte Carlo replicates per effect-size setting (1000)
- Significance threshold alpha=0.05 for declaring differential abundance
- R environment with metamicrobiomeR and the BEZI gamlss family

## Outputs

- Power-vs-effect-size table (CSV) with empirical power for GAMLSS-BEZI and LMAS at mu2 = 0.4, 0.3, 0.2
- Power curve figure (ggplot2) replicating Fig 1b

## How to apply

Generate synthetic two-group relative abundance data from a zero-inflated beta distribution using gamlss' BEZI family with mu1=0.5, zero-inflation alpha=0.5, precision phi=5, and n=100 samples per group, sweeping mu2 through 0.4, 0.3, and 0.2 to grade effect size. For each of 1000 Monte Carlo replicates per setting, test differential abundance with GAMLSS-BEZI via metamicrobiomeR's gamlss-based comparison (as implemented in taxa.compare with propmed.rel='gamlss') and with the LMAS linear model applied to arcsin square-root transformed proportions. Compute empirical power at each effect size as the proportion of replicates with p<0.05 for each method. The rationale is that LM on transformed data 'does not address the inflation of zero values', whereas GAMLSS-BEZI 'directly address[es] the distribution of microbiome relative abundance data', so BEZI is expected to be more powerful, with power monotonically increasing as mu2 decreases from 0.4 to 0.2.

## Related tools

- **metamicrobiomeR** (Provides the GAMLSS-BEZI differential abundance testing arm of the power simulation via its gamlss-based comparison workflow (taxa.compare with propmed.rel='gamlss')) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Zero-inflated beta distribution used both to generate the synthetic simulation data (mu, alpha, phi parameters) and to fit the more powerful test arm) — https://github.com/nhanhocu/metamicrobiomeR
- **MaAsLin** (Source of the comparison-method rationale: LMAS applies arcsin square-root transformation of relative abundance data before linear modeling)
- **R** (Implementation language for the simulation loop, metamicrobiomeR analyses, and ggplot2 power-curve rendering (R 3.4.2 or higher required)) — https://github.com/nhanhocu/metamicrobiomeR

## Examples

```
install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR)
```

## Evaluation signals

- Empirical power increases monotonically with effect size (mu2 from 0.4 to 0.2) for both methods
- GAMLSS-BEZI empirical power exceeds LMAS power at each of the three mu2 settings (0.4, 0.3, 0.2)
- Power values are bounded in [0,1] and, at 1000 replicates, Monte Carlo standard error is at most ~sqrt(0.25/1000) ≈ 0.016
- Simulation data exhibit zero inflation consistent with alpha=0.5 (proportion of zeros near 50%) and beta spread consistent with phi=5
- The rendered figure and CSV agree on the same power estimates for both methods across effect sizes

## Limitations

- Simulation covers only a single bacterial species per setting; results may not generalize to taxa-specific distribution shapes
- Power conclusions are conditional on the BEZI parameterization chosen (mu1=0.5, alpha=0.5, phi=5, n=100 per group); other prevalence or dispersion regimes may yield different rankings
- Linear models (including LMAS) do not address the zero-inflated beta nature of relative abundance data, so LMAS serves as a baseline rather than an optimal comparator

## Evidence

- [abstract] Simulation finding support: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [intro] Baseline method limitation: "However, this adjustment does not address the inflation of zero values in microbiome relative abundance data."
- [intro] Distributional rationale: "these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [methods] Tool installation: "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR)"
- [readme] Paper and simulations pointer: "The published paper describing the methodology as well as simulations and example analyses/results can be found at BMC Bioinformatics."
