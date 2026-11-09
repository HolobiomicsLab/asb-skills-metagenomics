---
name: mcmc-posterior-inference
description: Use when posterior samples of covariate-taxon or mediation-effect inclusion
  are needed from compositional count data via MicroBVS MCMC (DMbvs_R, MCMC_Med) before
  applying selected_DM or Selection_Med1/Med2.
when_to_use_negative:
- Input is non-compositional (e.g. raw feature tables without a compositional/DM or
  DTM structure) or lacks covariates whose association with taxa/mediation is the
  target
- You need per-covariate frequentist testing rather than Bayesian posterior inclusion
  probabilities — this skill is specific to MicroBVS's MCMC + MPPI framework
- Data has a known phylogenetic tree structure and you need graph-based inclusion
  priors with branch-level inference (232 dietary factor-branch associations case
  study), which goes beyond the plain MCMC posterior-inference workflow here
tools:
- name: MicroBVS
  role: R package providing DMbvs_R, MCMC_Med, selected_DM, Selection_Med1/Selection_Med2
    and Simulate_MedDM for MCMC posterior sampling and variable selection
  repo: https://github.com/mkoslovsky/MicroBVS
- name: Rcpp
  role: Required dependency — main MCMC functions operate in C++ via Rcpp wrappers
  repo: https://github.com/mkoslovsky/MicroBVS
- name: RcppArmadillo
  role: Required dependency for the C++ linear-algebra backends of the samplers
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Runtime environment in which the wrapper functions are sourced and posterior
    summaries/plots are produced
  repo: https://github.com/mkoslovsky/MicroBVS
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:c1517774f95ac6dc3b016394effe5df9ba114049f39ef8839c6a8fdf660c4ed3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mkoslovsky/MicroBVS
  tools:
  - MicroBVS
  - R
  - Rcpp
  - RcppArmadillo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mcmc-posterior-inference

## Summary

Run MCMC posterior sampling for Bayesian variable selection on compositional microbiome data using MicroBVS — Dirichlet-multinomial (DMbvs_R) and Dirichlet-tree multinomial (MCMC_Med) samplers whose posterior samples feed MPPI-based selection at a 0.5 threshold.

## When to use

Use when posterior samples of covariate-taxon or mediation-effect inclusion are needed from compositional count data via MicroBVS MCMC (DMbvs_R, MCMC_Med) before applying selected_DM or Selection_Med1/Med2.

## When NOT to use

- Input is non-compositional (e.g. raw feature tables without a compositional/DM or DTM structure) or lacks covariates whose association with taxa/mediation is the target
- You need per-covariate frequentist testing rather than Bayesian posterior inclusion probabilities — this skill is specific to MicroBVS's MCMC + MPPI framework
- Data has a known phylogenetic tree structure and you need graph-based inclusion priors with branch-level inference (232 dietary factor-branch associations case study), which goes beyond the plain MCMC posterior-inference workflow here

## Inputs

- Simulated or observed compositional microbiome count data from simulate_DM (e.g. n_obs=100, n_taxa=75, n_vars=30, rho=0.3, seed=1)
- Simulated mediation data from Simulate_MedDM (e.g. n_obs=100, n_taxa=20, seed=123) with known true mediator taxa
- Prior specification: prior='BB' (Beta-Binomial) with a=1, b=9 for DMbvs_R; MPPI threshold 0.5; burn-in 1000 iterations

## Outputs

- Posterior samples of mediation effects / inclusion indicators for all taxa from MCMC_Med
- Selected covariate-taxon pairs table from selected_DM (e.g. written to selected_pairs.csv)
- Selected mediators, MPPIs, and 95% credible intervals (e.g. written to mediation_selection_results.csv or selection_med2_results.csv)

## How to apply

In R, install MicroBVS from GitHub via devtools::install_github('mkoslovsky/MicroBVS', build_vignettes=TRUE), which requires Rcpp and RcppArmadillo since the main functions operate in C++. For covariate association with compositional counts, call simulate_DM (e.g. n_obs=100, n_taxa=75, n_vars=30, rho=0.3, seed=1) then fit DMbvs_R with prior='BB', a=1, b=9, discarding the first 1000 MCMC iterations as burn-in. For mediation of an exposure-outcome relationship, call Simulate_MedDM (n_obs=100, n_taxa=20, seed=123) and run MCMC_Med either per taxon (marginal strategy) or in a single joint run over all 20 taxa. From the posterior samples compute the marginal posterior probability of inclusion (MPPI) and apply selection at an MPPI threshold of 0.5 via selected_DM or Selection_Med1/Selection_Med2. Note the README documents a revised joint-model sampler in which the previous Gibbs update of auxiliary parameters c_ij has been replaced with a MH update, increasing computation time.

## Related tools

- **MicroBVS** (R package providing DMbvs_R, MCMC_Med, selected_DM, Selection_Med1/Selection_Med2 and Simulate_MedDM for MCMC posterior sampling and variable selection) — https://github.com/mkoslovsky/MicroBVS
- **Rcpp** (Required dependency — main MCMC functions operate in C++ via Rcpp wrappers) — https://github.com/mkoslovsky/MicroBVS
- **RcppArmadillo** (Required dependency for the C++ linear-algebra backends of the samplers) — https://github.com/mkoslovsky/MicroBVS
- **R** (Runtime environment in which the wrapper functions are sourced and posterior summaries/plots are produced) — https://github.com/mkoslovsky/MicroBVS

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- DMbvs_R selections at MPPI > 0.5 (burn-in 1000) recover the 4 covariate-taxon associations embedded by simulate_DM in the 100x75x30 dataset, and selected_pairs.csv exists with a row per selected pair
- Strategy 1 (per-taxon MCMC_Med + Selection_Med1, MPPI threshold 0.5) lists taxa 3 and 4 as relative mediators while taxon 5 is not selected, matching the vignette outcome
- Strategy 2 (single MCMC_Med run + Selection_Med2 on all taxa) selects only taxon 3, missing taxa 4 and 5, checked field-by-field against the expected outcome record
- Output CSVs contain the required schema fields (taxon index, selected/not-selected flag, MPPI values, 95% credible intervals)
- Posterior selection should align with the simulated ground truth (e.g. taxa 3, 4, 5 active in Simulate_MedDM default setting)

## Limitations

- Convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model
- No software version numbers are provided for MicroBVS or its dependencies, and there is no changelog or archival (e.g. Zenodo) DOI for the code — only the GitHub URL
- The revised joint-model sampler (MH update of c_ij) increases computation time relative to the original Gibbs update; partial updates via a 'rate' parameter (typically >= 0.50) are needed to speed it up
- Selection can miss true mediators: Strategy 1 failed to identify the 5th true mediator taxon, and joint Strategy 2 missed both the 4th and 5th

## Evidence

- [abstract] abstract: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection, for the identification of covariates associated with microbial taxa abundance data"
- [results] results: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
- [discussion] discussion: "Other requirements: R Rcpp RcppArmadillo ape MCMCpack mvtnorm ggplot2 GGMselect devtools ape igraph"
- [readme] readme: "The main functions operate in C++ via the R package Rcpp. These functions can be sourced by a set of wrapper functions that enable easy implementation of the code in the R environment."
