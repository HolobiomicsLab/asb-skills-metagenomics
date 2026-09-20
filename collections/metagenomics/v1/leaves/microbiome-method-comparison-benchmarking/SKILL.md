---
name: microbiome-method-comparison-benchmarking
description: Use when benchmarking differential-abundance methods for zero-inflated
  microbiome relative abundance data, e.g. to quantify empirical power of GAMLSS-BEZI
  vs. transformed linear models across effect sizes.
when_to_use_negative:
- Input is a real taxon relative-abundance table for a substantive comparison rather
  than simulated BEZI data — use the standard taxa.compare / meta.taxa analysis workflow
  instead
- The target method addresses zero inflation through a non-beta model (e.g. zero-inflated
  Gaussian or compositional methods like ALDEx2/ANCOM) not implemented in metamicrobiomeR
  — this benchmark harness only covers GAMLSS-BEZI vs. LMAS
- The goal is meta-analysis pooling across studies (meta.taxa) rather than within-study
  differential-abundance power comparison
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metamicrobiomeR
  role: Implements the GAMLSS-BEZI differential relative abundance test (taxa.compare
    with propmed.rel='gamlss') that serves as the candidate method under benchmarking
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Zero-inflated beta GAMLSS family that directly models the zero-inflated beta
    distribution of microbiome relative abundance data; the method whose power is
    being estimated
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: MaAsLin
  role: 'Reference competitor approach: linear model applied to arcsin square-root-transformed
    relative abundances (LMAS baseline in the power simulation)'
- name: R
  role: Language in which simulations, both statistical tests, power computation (proportion
    of replicates with p<0.05), and the ggplot2 Fig 1b replication are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:ee30e1909e9e854f93d2f054ac39bf38c78030804cf69d54befe6ff4ebe93a55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - MaAsLin
  - R
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# microbiome-method-comparison-benchmarking

## Summary

Simulation-based benchmarking skill that compares statistical power of GAMLSS-BEZI (via the metamicrobiomeR R package) against transformation-based linear modeling (MaAsLin-style arcsin square root, LMAS) for detecting differential relative abundances, replicating the paper's Fig 1b power curves across effect sizes.

## When to use

Use when benchmarking differential-abundance methods for zero-inflated microbiome relative abundance data, e.g. to quantify empirical power of GAMLSS-BEZI vs. transformed linear models across effect sizes.

## When NOT to use

- Input is a real taxon relative-abundance table for a substantive comparison rather than simulated BEZI data — use the standard taxa.compare / meta.taxa analysis workflow instead
- The target method addresses zero inflation through a non-beta model (e.g. zero-inflated Gaussian or compositional methods like ALDEx2/ANCOM) not implemented in metamicrobiomeR — this benchmark harness only covers GAMLSS-BEZI vs. LMAS
- The goal is meta-analysis pooling across studies (meta.taxa) rather than within-study differential-abundance power comparison

## Inputs

- Synthetic BEZI-distributed relative abundance matrices for two groups (mu1=0.5, mu2 in {0.4, 0.3, 0.2}; alpha=0.5; phi=5; n=100 per group)
- Group labels (two-level comparison factor, e.g. gender/breastfeeding-style binary variable)
- R environment with metamicrobiomeR, gamlss, and ggplot2 packages

## Outputs

- Power-vs-effect-size table (CSV) listing empirical power for GAMLSS-BEZI and LMAS at mu2 = 0.4, 0.3, 0.2
- Power curve figure replicating Fig 1b (ggplot2), one curve per method
- Per-replicate p-values from GAMLSS-BEZI (taxa.compare) and LMAS tests for each simulation setting

## How to apply

Generate synthetic two-group relative abundance data from a zero-inflated beta (BEZI) distribution with mu1=0.5, zero-inflation alpha=0.5, precision phi=5, and n=100 samples per group, varying mu2 (0.4, 0.3, 0.2) to create three effect-size settings. For each of 1000 replicates per setting, test differential abundance twice: with GAMLSS-BEZI via metamicrobiomeR (propmed.rel='gamlss' in taxa.compare) and with a MaAsLin-style linear model on arcsin square-root-transformed abundances (LMAS). Compute empirical power for each method/effect size as the proportion of replicates with p<0.05. Assemble a power-vs-effect-size CSV table and render a ggplot2 power curve replicating Fig 1b. The rationale, grounded in the paper, is that the arcsin square-root transformation used by MaAsLin 'does not address the inflation of zero values', so the zero-inflated beta GAMLSS should show higher power across all simulated effect sizes.

## Related tools

- **metamicrobiomeR** (Implements the GAMLSS-BEZI differential relative abundance test (taxa.compare with propmed.rel='gamlss') that serves as the candidate method under benchmarking) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Zero-inflated beta GAMLSS family that directly models the zero-inflated beta distribution of microbiome relative abundance data; the method whose power is being estimated) — https://github.com/nhanhocu/metamicrobiomeR
- **MaAsLin** (Reference competitor approach: linear model applied to arcsin square-root-transformed relative abundances (LMAS baseline in the power simulation))
- **R** (Language in which simulations, both statistical tests, power computation (proportion of replicates with p<0.05), and the ggplot2 Fig 1b replication are implemented)

## Examples

```
install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR); taxacom<-taxa.compare(taxtab=sim_taxtab,propmed.rel="gamlss",comvar="group",longitudinal="no",p.adjust.method="fdr")
```

## Evaluation signals

- Empirical power must increase monotonically with effect size (mu2 decreasing from 0.4 to 0.2 means power should rise as the gap from mu1=0.5 widens) for both methods
- GAMLSS-BEZI empirical power must exceed LMAS power at each of the three mu2 settings (0.4, 0.3, 0.2), consistent with the paper's reported simulation findings
- Power table (CSV) must contain one row per method per effect size with power estimated from 1000 replicates each
- Rendered figure must visually reproduce the paper's Fig 1b power-curve layout with two method curves over three effect-size points
- Simulation parameters must match the design: mu1=0.5, alpha=0.5, phi=5, n=100 per group, 1000 replicates, significance threshold p<0.05

## Limitations

- The arcsin square-root transformation baseline (LMAS) does not address zero inflation, so the power advantage may be specific to zero-inflated beta-distributed simulated data
- The simulation covers one bacterial species and three effect-size settings only; results may not generalize to other taxa, sample sizes, or covariance structures
- The GitHub version differs from the CRAN version of metamicrobiomeR (minor changes), which could affect reproducibility of exact power numbers

## Evidence

- [abstract] GAMLSS-BEZI performs well in simulation-based power testing: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [intro] MaAsLin arcsin square root transformation as comparison baseline: "Transformations (e.g. arcsin square root) of relative abundance data to make it resemble continuous data to use in LM has been proposed by Morgan et al. (implemented in MaAsLin software)"
- [intro] Zero-inflation not addressed by transformation: "However, this adjustment does not address the inflation of zero values in microbiome relative abundance data."
- [intro] Rationale for zero-inflated beta modeling: "these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [methods] Package installation for running the benchmark: "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR)"
- [readme] Simulations and results available in published paper: "The published paper describing the methodology as well as simulations and example analyses/results can be found at BMC Bioinformatics."
