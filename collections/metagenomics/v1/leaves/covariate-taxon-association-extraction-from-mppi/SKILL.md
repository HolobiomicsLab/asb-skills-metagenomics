---
name: covariate-taxon-association-extraction-from-mppi
description: Use when you have MCMC posterior samples from a Dirichlet-multinomial
  (DM) regression fit in MicroBVS and need to declare which covariate-taxon associations
  are selected, e.g. against a simulated ground truth.
when_to_use_negative:
- Your goal is phenotypic outcome prediction rather than variable selection — MicroBVS
  also ships a separate joint model for simultaneous identification of covariate associations
  and prediction of phenotypic outcomes
- The data are not compositional taxa counts (or other compositional data) paired
  with a covariate matrix — the DM/DTM likelihoods in MicroBVS are defined for abundance/compositional
  data
- You need phylogenetic-structure-aware inference with a known tree — the DM formulation
  used here does not itself encode tree-like structure; use the DTM (Dirichlet-tree
  multinomial) variant of the package instead
tools:
- name: MicroBVS
  role: Provides simulate_DM, DMbvs_R, and selected_DM functions used to simulate
    the DM dataset, fit the variable-selection model, and extract selected covariate-taxon
    pairs at an MPPI threshold
  repo: github.com/mkoslovsky/MicroBVS
- name: Rcpp
  role: Enables the C++ back-end for MicroBVS's main functions, called through R wrapper
    functions; required dependency to install before using the package
  repo: github.com/mkoslovsky/MicroBVS
- name: RcppArmadillo
  role: Required R dependency for compiling and running MicroBVS's C++ MCMC routines
    underlying DMbvs_R
  repo: github.com/mkoslovsky/MicroBVS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:5fefebd2f11500689cc450e034a8553b3172146b13e192a73c12aa61f238b4f7
schema_version: 0.2.0
metadata:
  license_tier: open
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
n_claims: 14
---

# covariate-taxon-association-extraction-from-mppi

## Summary

Extracts selected covariate-taxon associations from Dirichlet-multinomial Bayesian variable selection fits (DMbvs_R in MicroBVS) by thresholding the marginal posterior probability of inclusion (MPPI), providing ground-truth-comparable sets of covariate-taxon pairs.

## When to use

Use when you have MCMC posterior samples from a Dirichlet-multinomial (DM) regression fit in MicroBVS and need to declare which covariate-taxon associations are selected, e.g. against a simulated ground truth.

## When NOT to use

- Your goal is phenotypic outcome prediction rather than variable selection — MicroBVS also ships a separate joint model for simultaneous identification of covariate associations and prediction of phenotypic outcomes
- The data are not compositional taxa counts (or other compositional data) paired with a covariate matrix — the DM/DTM likelihoods in MicroBVS are defined for abundance/compositional data
- You need phylogenetic-structure-aware inference with a known tree — the DM formulation used here does not itself encode tree-like structure; use the DTM (Dirichlet-tree multinomial) variant of the package instead

## Inputs

- Posterior MCMC samples from DMbvs_R (Dirichlet-multinomial regression with Bayesian variable selection)
- Compositional taxa count matrix (e.g. simulated via simulate_DM with n_obs, n_taxa, n_vars parameters)
- Covariate design matrix (e.g. 30 covariates for 100 subjects)

## Outputs

- Selected covariate-taxon pairs table (e.g. selected_pairs.csv with one row per pair passing the MPPI threshold)
- Marginal posterior probability of inclusion (MPPI) values per covariate-taxon pair

## How to apply

Simulate or load compositional count data (e.g. simulate_DM with n_obs=100, n_taxa=75, n_vars=30, rho=0.3) and fit DMbvs_R with a Beta-Binomial inclusion prior (prior='BB', a=1, b=9), discarding the first 1000 MCMC iterations as burn-in. Pass the retained posterior samples to selected_DM with an MPPI threshold of 0.5: taxa-covariate pairs whose marginal posterior probability of inclusion exceeds 0.5 are declared selected. Write the selected pairs table (one row per selected covariate-taxon pair) to a file such as selected_pairs.csv. Rationale: the MPPI threshold turns posterior inclusion probabilities into a discrete selection that can be compared against embedded ground-truth associations — in the vignette benchmark, 4 covariates truly associated with 4 of 75 simulated taxa out of 30x75 potential pairs.

## Related tools

- **MicroBVS** (Provides simulate_DM, DMbvs_R, and selected_DM functions used to simulate the DM dataset, fit the variable-selection model, and extract selected covariate-taxon pairs at an MPPI threshold) — github.com/mkoslovsky/MicroBVS
- **Rcpp** (Enables the C++ back-end for MicroBVS's main functions, called through R wrapper functions; required dependency to install before using the package) — github.com/mkoslovsky/MicroBVS
- **RcppArmadillo** (Required R dependency for compiling and running MicroBVS's C++ MCMC routines underlying DMbvs_R) — github.com/mkoslovsky/MicroBVS

## Examples

```
library(MicroBVS); sim <- simulate_DM(n_obs=100, n_taxa=75, n_vars=30, rho=0.3, seed=1); fit <- DMbvs_R(sim, prior="BB", a=1, b=9, burnin=1000); sel <- selected_DM(fit, threshold=0.5)
```

## Evaluation signals

- Selected pairs at MPPI > 0.5 exactly recover the 4 covariate-taxon associations embedded in the simulate_DM simulation (n_obs=100, n_taxa=75, n_vars=30, rho=0.3, seed=1)
- selected_pairs.csv exists and contains one row for each selected covariate-taxon pair
- The first 1000 MCMC iterations were discarded as burn-in before computing inclusion probabilities
- Inclusion prior parameters match the specified Beta-Binomial setting (prior='BB', a=1, b=9)
- No more than the expected number of pairs are selected out of the 30 x 75 potential covariate-taxon associations

## Limitations

- Convergence of the MCMC sampler is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model, so MPPI-based selection may be unstable for correlated or very dense settings
- No software version numbers are reported for MicroBVS or its dependencies, and the repository has no changelog or archival (e.g. Zenodo) record, complicating reproducibility of the selection step
- Model space grows combinatorially — for B branches and P covariates there are 2^(B×P) potential models — making exhaustive verification of selection infeasible for large settings

## Evidence

- [intro] MicroBVS identifies covariate-taxon associations via Bayesian variable selection: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection, for the identification of covariates associated with microbial taxa abundance data"
- [readme] The main MicroBVS functions run in C++ via Rcpp wrappers: "The main functions operate in C++ via the R package Rcpp. These functions can be sourced by a set of wrapper functions that enable easy implementation of the code in the R environment."
- [readme] MicroBVS installation from GitHub with vignettes: "Then to install the MicroBVS package, simply run 

library(devtools)
install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
- [results] Graph-based inclusion priors can be specified in MicroBVS: "the MicroBVS package can be specified with graph-based inclusion priors, similar to [26, 27]. See the vignette for details"
