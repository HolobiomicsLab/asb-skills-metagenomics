---
name: zero-inflated-beta-distribution-simulation
description: Use when benchmarking differential abundance methods on zero-inflated
  relative abundance data — e.g., estimating empirical type I error rates at alpha
  0.01/0.05/0.1 or power across effect sizes via Monte Carlo replicates.
when_to_use_negative:
- Input is a real observed microbiome relative abundance table (e.g., taxtab from
  QIIME output) rather than data needing synthetic generation — use taxa.compare-based
  analysis instead
- The outcome data are counts or compositional absolute abundances rather than relative
  abundances resembling a zero-inflated beta distribution
- The goal is cross-study meta-analysis pooling of estimates rather than within-study
  method performance evaluation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
tools:
- name: metamicrobiomeR
  role: Provides GAMLSS-BEZI fitting for between-group comparison of simulated relative
    abundances; installed from GitHub and loaded in the simulation workflow
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: gamlss
  role: Supplies the GAMLSS framework and BEZI zero-inflated beta distribution used
    to generate and model simulated data
- name: MaAsLin
  role: 'Reference LMAS approach: linear model applied to arcsin square root-transformed
    relative abundances, used as the power/type-I-error comparator'
- name: devtools
  role: Used to install metamicrobiomeR from GitHub (install_github) before running
    simulations
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:aec015b5c16fe95f75a0ade9932499574f8eb573cd3c22dc0efd6e503e01ff48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - MaAsLin
  - R
  - devtools
  - gamlss
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# zero-inflated-beta-distribution-simulation

## Summary

Generates synthetic microbiome relative abundance data from a zero-inflated beta (BEZI) distribution and evaluates differential-abundance methods (GAMLSS-BEZI vs LMAS) via Monte Carlo simulation of type I error and power, as done in the metamicrobiomeR paper.

## When to use

Use when benchmarking differential abundance methods on zero-inflated relative abundance data — e.g., estimating empirical type I error rates at alpha 0.01/0.05/0.1 or power across effect sizes via Monte Carlo replicates.

## When NOT to use

- Input is a real observed microbiome relative abundance table (e.g., taxtab from QIIME output) rather than data needing synthetic generation — use taxa.compare-based analysis instead
- The outcome data are counts or compositional absolute abundances rather than relative abundances resembling a zero-inflated beta distribution
- The goal is cross-study meta-analysis pooling of estimates rather than within-study method performance evaluation

## Inputs

- Simulation parameter settings (mu1, mu2, alpha/zero-inflation, phi/precision, group sample size n, number of Monte Carlo replicates)
- Nominal alpha levels for type I error evaluation (0.01, 0.05, 0.1)
- Effect-size settings for power evaluation (mu2 = 0.4, 0.3, 0.2)

## Outputs

- Empirical type I error rate table (rows = sample sizes n=10/100/500, columns = alpha 0.01/0.05/0.1), e.g. type1_error_rates.csv
- Empirical power table by method and effect size
- Power-vs-effect-size figure replicating Fig 1b (ggplot2)

## How to apply

Simulate BEZI-distributed relative abundance data with explicit parameters: null settings use mu1=mu2=0.5, zero-inflation alpha1=alpha2=0.5, precision phi=5, at sample sizes n=10, 100, and 500; power settings vary mu2 (0.4, 0.3, 0.2) against mu1=0.5 with n=100 per group, running 1000 Monte Carlo replicates per setting. For each replicate, test between-group differential abundance with GAMLSS-BEZI via metamicrobiomeR (gamlss BEZI family) and with the LMAS comparison (linear model on arcsin square root-transformed relative abundances, the MaAsLin-style approach). Count null rejections at alpha levels 0.01, 0.05, and 0.1 to compute empirical type I error rates, and proportions of p<0.05 under alternatives to compute empirical power. Assemble results into tables (rows = sample sizes or effect sizes) and render power curves (e.g., replicating Fig 1b with ggplot2). The rationale, per the paper, is that t-tests, Wilcoxon tests, and linear models do not address the zero-inflated beta distribution of relative abundances, whereas GAMLSS-BEZI directly models it.

## Related tools

- **metamicrobiomeR** (Provides GAMLSS-BEZI fitting for between-group comparison of simulated relative abundances; installed from GitHub and loaded in the simulation workflow) — https://github.com/nhanhocu/metamicrobiomeR
- **gamlss** (Supplies the GAMLSS framework and BEZI zero-inflated beta distribution used to generate and model simulated data)
- **MaAsLin** (Reference LMAS approach: linear model applied to arcsin square root-transformed relative abundances, used as the power/type-I-error comparator)
- **devtools** (Used to install metamicrobiomeR from GitHub (install_github) before running simulations)

## Examples

```
devtools::install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR); y <- gamlss.dist::rBEZI(n=100, mu=0.5, sigma=0.5, nu=5); fit <- gamlss(y ~ group, family=BEZI)
```

## Evaluation signals

- Each empirical type I error rate lies within Monte Carlo tolerance (~2 binomial standard errors over 1000 replicates) of its nominal level 0.01/0.05/0.1, reproducing the well-controlled pattern in Table 1
- Empirical power increases monotonically with effect size (mu2 = 0.4 > 0.3 > 0.2) for both GAMLSS-BEZI and LMAS
- GAMLSS-BEZI power exceeds LMAS power at each of the three effect-size settings, consistent with Fig 1b
- Results table has the expected structure (rows = sample sizes or effect sizes, columns = alpha levels or methods) and is exported as CSV
- All 1000 replicates complete for each parameter setting with no missing p-values before rejection counts are computed

## Limitations

- The tutorial implementing the simulation workflow is in Additional file 1, which is referenced but not inline-accessible in the article text
- Simulations cover only one bacterial species per replicate and specific BEZI parameter settings (mu, alpha=0.5, phi=5), so conclusions may not generalize to other zero-inflation/precision regimes
- The GitHub version has minor changes compared to the CRAN version; the README recommends using the CRAN version of metamicrobiomeR

## Evidence

- [abstract] Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies.: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [intro] Zero-inflated beta distribution rationale: "these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [readme] BEZI implementation in metamicrobiomeR: "The *metamicrobiomeR* package implements Generalized Additive Model for Location, Scale and Shape (GAMLSS) 
    with zero inflated beta (BEZI) family for analysis of microbiome relative abundance data"
- [intro] LMAS comparator rationale: "Transformations (e.g. arcsin square root) of relative abundance data to make it resemble continuous data to use in LM has been proposed by Morgan et al."
- [methods] Package installation method: "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR)"
