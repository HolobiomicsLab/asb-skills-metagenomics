---
name: power-curve-visualization
description: Use when evaluating or visualizing statistical power of differential
  abundance methods on zero-inflated relative abundance data across effect sizes,
  e.g. comparing GAMLSS-BEZI against transformation-based linear models via simulation.
when_to_use_negative:
- Input is real observed abundance data rather than simulated BEZI data — this skill
  estimates empirical power from known ground-truth effect sizes, which observed data
  do not provide
- The goal is meta-analysis pooling of estimates across studies (use metamicrobiomeR's
  random-effects meta.taxa workflow instead)
- The comparison methods model count/COMPOSITIONAL data with their own variance handling
  (e.g. ANCOM-style), not transformed or beta-distributed relative abundances as in
  the GAMLSS-BEZI vs LMAS contrast
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: metamicrobiomeR
  role: Provides the GAMLSS-BEZI differential abundance test (taxa.compare / gamlss
    BEZI family) whose power is estimated in the simulation
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Programming language for the simulation loop (1000 replicates), power computation,
    and CSV table assembly (R 3.4.2 or higher required)
- name: GAMLSS-BEZI
  role: Zero-inflated beta distribution used both to generate synthetic data (mu1,
    mu2, alpha, phi) and as the tested statistical model
- name: MaAsLin
  role: 'Source of the comparison method: arcsin square-root transformation of relative
    abundances feeding a linear model (LMAS)'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:94bc86e491b01554859fa9a37ee74ace4cac0c5235f0b630aef8ae564e0702ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - MaAsLin
  - R
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# power-curve-visualization

## Summary

Generates empirical power-vs-effect-size curves for differential relative abundance tests by simulating zero-inflated beta (BEZI) microbiome data and plotting the proportion of replicates with p<0.05 for each method, replicating the metamicrobiomeR paper's Fig 1b comparison of GAMLSS-BEZI vs the MaAsLin-style arcsin square-root linear model (LMAS).

## When to use

Use when evaluating or visualizing statistical power of differential abundance methods on zero-inflated relative abundance data across effect sizes, e.g. comparing GAMLSS-BEZI against transformation-based linear models via simulation.

## When NOT to use

- Input is real observed abundance data rather than simulated BEZI data — this skill estimates empirical power from known ground-truth effect sizes, which observed data do not provide
- The goal is meta-analysis pooling of estimates across studies (use metamicrobiomeR's random-effects meta.taxa workflow instead)
- The comparison methods model count/COMPOSITIONAL data with their own variance handling (e.g. ANCOM-style), not transformed or beta-distributed relative abundances as in the GAMLSS-BEZI vs LMAS contrast

## Inputs

- Simulation settings: BEZI parameters (mu1=0.5, mu2 in {0.4, 0.3, 0.2}, alpha=0.5, phi=5), n=100 per group, 1000 replicates per effect size
- Per-replicate p-values from GAMLSS-BEZI (metamicrobiomeR taxa.compare / gamlss BEZI family)
- Per-replicate p-values from LMAS (arcsin square-root-transformed linear model)

## Outputs

- Power-vs-effect-size table (CSV) with empirical power (proportion of p<0.05 replicates) per method per mu2 setting
- Power curve figure (Fig 1b replica) rendered with ggplot2
- Validation that power increases monotonically with effect size and GAMLSS-BEZI power exceeds LMAS power at mu2 = 0.4, 0.3, and 0.2

## How to apply

Generate synthetic two-group relative abundance data from a zero-inflated beta distribution using the BEZI parameterization with mu1=0.5, zero-inflation alpha=0.5, precision phi=5, and n=100 samples per group, varying mu2 across effect settings (0.4, 0.3, 0.2). For each of 1000 replicates per setting, test differential abundance with GAMLSS-BEZI via metamicrobiomeR and with the MaAsLin-style arcsin square-root-transformed linear model (LMAS). Compute empirical power for each method and effect size as the proportion of replicates with p<0.05. Assemble a power-vs-effect-size table (CSV) and render the power curve with ggplot2, mirroring Fig 1b of the paper. The rationale is that GAMLSS-BEZI directly addresses the zero-inflated beta distribution of relative abundances, while the arcsin square-root transformation used by MaAsLin does not address the inflation of zero values, so the curve should show GAMLSS-BEZI power exceeding LMAS at every effect size.

## Related tools

- **metamicrobiomeR** (Provides the GAMLSS-BEZI differential abundance test (taxa.compare / gamlss BEZI family) whose power is estimated in the simulation) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Programming language for the simulation loop (1000 replicates), power computation, and CSV table assembly (R 3.4.2 or higher required))
- **GAMLSS-BEZI** (Zero-inflated beta distribution used both to generate synthetic data (mu1, mu2, alpha, phi) and as the tested statistical model)
- **MaAsLin** (Source of the comparison method: arcsin square-root transformation of relative abundances feeding a linear model (LMAS))

## Examples

```
install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR); library(gamlss); sim <- rBEZI(n=100, mu=0.5, sigma=5, nu=0.5)
```

## Evaluation signals

- Power increases monotonically with effect size for both methods (mu2 = 0.4 > 0.3 > 0.2)
- GAMLSS-BEZI empirical power exceeds LMAS power at each of the three mu2 settings (0.4, 0.3, 0.2), consistent with the paper's reported simulation findings
- Each power estimate is based on 1000 replicates with n=100 per group, alpha=0.5, phi=5, mu1=0.5
- Power is computed as the proportion of replicates with p<0.05 per method per effect size
- CSV table and ggplot2 figure agree on the same power values for every (method, effect size) pair

## Limitations

- Simulation covers only one bacterial species and three effect-size settings; conclusions may not generalize to all taxa or effect sizes
- LMAS (arcsin square-root transformation) does not address the inflation of zero values in relative abundance data, which is the source of its power deficit rather than a general flaw of linear models
- Results depend on the chosen BEZI parameters (alpha=0.5 zero inflation, phi=5 precision); different zero-inflation or precision regimes may shift the power curves

## Evidence

- [results] GAMLSS-BEZI achieves higher power than LMAS across effect sizes: [evidence span withheld — anchor status: not_found]
- [discussion] GAMLSS-BEZI addresses the distribution of relative abundance data: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [intro] Arcsin square-root transformation does not address zero inflation: "However, this adjustment does not address the inflation of zero values in microbiome relative abundance data."
- [intro] MaAsLin uses arcsin square-root transformation with linear models: "Transformations (e.g. arcsin square root) of relative abundance data to make it resemble continuous data to use in LM has been proposed by Morgan et al. (implemented in MaAsLin software)"
- [readme] metamicrobiomeR is available via GitHub and applies GAMLSS with BEZI family: "The *metamicrobiomeR* package implements Generalized Additive Model for Location, Scale and Shape (GAMLSS) 
    with zero inflated beta (BEZI) family for analysis of microbiome relative abundance data"
