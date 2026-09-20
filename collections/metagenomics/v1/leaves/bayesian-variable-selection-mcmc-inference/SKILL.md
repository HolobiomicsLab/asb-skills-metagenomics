---
name: bayesian-variable-selection-mcmc-inference
description: Use when relating a covariate matrix (e.g., dietary or clinical variables)
  to compositional taxa count data that have a known phylogenetic tree, and the goal
  is sparse branch-by-covariate association selection with uncertainty quantification.
when_to_use_negative:
- The compositional data have no known tree-like structure and a non-phylogenetic
  parameterization is preferred — although MicroBVS can handle data without a tree,
  this skill centers on the DTM tree-structured model
- The goal is penalized (frequentist) selection rather than Bayesian posterior inference
  — the article's case study shows the penalized approach identified 271 associations
  overall vs. 232 with the Bayesian DTM method
- The outcome is a phenotypic prediction or mediation effect rather than covariate-branch
  association selection — the repository's separate joint/mediation models are the
  appropriate entry points
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MicroBVS
  role: Provides DTMbvs_R (MCMC fitting with Beta-Binomial inclusion prior), simulate_DTM
    (data generation), and selected_DTM (MPPI >= 0.50 selection) for branch-by-covariate
    association identification
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Environment in which MicroBVS wrapper functions are called to run the C++
    samplers and perform posterior inference
- name: Rcpp
  role: Backend through which the main MCMC functions operate in C++ for the DTM Gibbs/variable-selection
    sampler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:50017e189a08339f5ec461439a8b227cf726a705c1d1de5dcf54a4c8e754263d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mkoslovsky/MicroBVS
  tools:
  - MicroBVS
  - R
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bayesian-variable-selection-mcmc-inference

## Summary

Fits Dirichlet-tree multinomial (DTM) regression with Bayesian variable selection via MCMC to identify covariate associations with phylogenetically structured compositional (microbiome) count data, then selects associations by marginal posterior probability of inclusion (MPPI) thresholding.

## When to use

Use when relating a covariate matrix (e.g., dietary or clinical variables) to compositional taxa count data that have a known phylogenetic tree, and the goal is sparse branch-by-covariate association selection with uncertainty quantification.

## When NOT to use

- The compositional data have no known tree-like structure and a non-phylogenetic parameterization is preferred — although MicroBVS can handle data without a tree, this skill centers on the DTM tree-structured model
- The goal is penalized (frequentist) selection rather than Bayesian posterior inference — the article's case study shows the penalized approach identified 271 associations overall vs. 232 with the Bayesian DTM method
- The outcome is a phenotypic prediction or mediation effect rather than covariate-branch association selection — the repository's separate joint/mediation models are the appropriate entry points

## Inputs

- Compositional taxa count matrix (subjects x taxa) with an accompanying phylogenetic tree (e.g., 5-leaf tree from simulate_DTM)
- Covariate/design matrix (e.g., 100 subjects x 50 covariates)
- Inclusion prior specification (Beta-Binomial with a=0.1, b=0.9)
- MCMC settings (burn-in iterations, e.g., 2,500) and MPPI selection threshold (e.g., 0.50)

## Outputs

- Per-branch, per-covariate MPPI posterior summaries from the DTM MCMC run
- Selected branch-by-covariate association set from selected_DTM at the MPPI >= 0.50 threshold
- Recovery counts (true positives vs. false positives vs. planted associations), e.g., written to dtmbvs_simulation_results.csv

## How to apply

Load the MicroBVS package (installed from GitHub with devtools::install_github) whose core samplers run in C++ via Rcpp. Simulate or supply DTM count data, e.g., simulate_DTM with 100 subjects, 50 covariates, a 5-leaf tree, and correlation rho=0.3. Fit the DTM model with DTMbvs_R using a Beta-Binomial inclusion prior (a=0.1, b=0.9), discarding the first 2,500 MCMC iterations as burn-in. Apply selected_DTM with an MPPI threshold of 0.50 to obtain the selected branch-by-covariate associations. Rationale: for B branches and P covariates the model space is 2^(B×P), so stochastic search over the tree structure with inclusion priors is needed; selection at MPPI >= 0.50 balances sensitivity against false positives (the vignette simulation recovers 9 of 15 planted associations with zero false positives).

## Related tools

- **MicroBVS** (Provides DTMbvs_R (MCMC fitting with Beta-Binomial inclusion prior), simulate_DTM (data generation), and selected_DTM (MPPI >= 0.50 selection) for branch-by-covariate association identification) — https://github.com/mkoslovsky/MicroBVS
- **R** (Environment in which MicroBVS wrapper functions are called to run the C++ samplers and perform posterior inference)
- **Rcpp** (Backend through which the main MCMC functions operate in C++ for the DTM Gibbs/variable-selection sampler)

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- On the vignette simulation (seed 1, rho=0.3, Beta-Binomial a=0.1/b=0.9, 2,500-iteration burn-in), selection at MPPI >= 0.50 recovers 9 of the 15 planted branch-by-covariate associations with zero false positives
- All reported MPPI values lie in [0, 1] and the selected set is a sparse subset of the 2^(B×P) possible branch-by-covariate models
- Convergence is plausible given the correlation structure between covariates and count data and the sparsity level — poor mixing under high rho or dense models invalidates the selection
- Recovery counts computed against the known planted associations match the target before results are written to dtmbvs_simulation_results.csv

## Limitations

- Convergence of the DTM MCMC is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model
- No software version numbers are provided for MicroBVS or its dependencies, and no DOI/Zenodo archival record exists — only the GitHub URL
- In the repository's related joint model, the auxiliary parameter c_ij Gibbs update was replaced with an MH update (as of 9/6/23), which increases computation time, mitigated by a 'full' variant or partial-update 'rate' parameter (>= 0.50 typically comparable in simulation)

## Evidence

- [abstract] MicroBVS package abstract claim: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection"
- [results] case study selection result: "We identified 232 dietary factor-branch associations with our Bayesian variable selection method for DTM regression models"
- [results] model-space motivation: "Specifically for B branches and P covariates, there are 2B×P potential models to choose from."
- [readme] installation via devtools: "install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
- [discussion] implementation stack: "Programming language: R and C++ Other requirements: R Rcpp RcppArmadillo ape MCMCpack mvtnorm ggplot2 GGMselect devtools ape igraph"
