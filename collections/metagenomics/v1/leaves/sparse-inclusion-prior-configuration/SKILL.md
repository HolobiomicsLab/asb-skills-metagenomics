---
name: sparse-inclusion-prior-configuration
description: Use when fitting DMbvs_R or DTM regression to compositional microbiome
  counts (e.g. 100 subjects, 75 taxa, 30 covariates) and you must set an inclusion
  prior (e.g. prior='BB', a=1, b=9) to encode sparsity.
when_to_use_negative:
- When the compositional data have a known phylogenetic tree and branch-level (DTM)
  selection is the target rather than per-taxon DM selection
- When the covariate space is dense by design (few covariates all expected relevant)
  — a Beta-Binomial sparse prior tuned for 30 covariates over 75 taxa will over-shrink
  the signal
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_2269
tools:
- name: MicroBVS
  role: R package providing simulate_DM, DMbvs_R, and selected_DM for Dirichlet-multinomial
    regression with Bayesian variable selection and configurable inclusion priors
  repo: https://github.com/mkoslovsky/MicroBVS
- name: Rcpp
  role: Dependency; the main DMbvs_R sampler functions operate in C++ via Rcpp and
    are wrapped by R functions
  repo: https://github.com/mkoslovsky/MicroBVS
- name: RcppArmadillo
  role: Dependency required for the C++ linear algebra underlying the MCMC sampler
  repo: https://github.com/mkoslovsky/MicroBVS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:cbc229e11315b9a36dbcb2949906a518b1a4bdc3b51e37933c97a651cbdb68db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mkoslovsky/MicroBVS
  tools:
  - MicroBVS
  - R
  - Rcpp
  - RcppArmadillo
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# Sparse inclusion prior configuration

## Summary

Configure the prior probabilities of covariate inclusion in MicroBVS Dirichlet(-tree) multinomial Bayesian variable selection — e.g. a Beta-Binomial prior with a=1, b=9 — so that the MCMC explores the sparse 2^(B×P) model space efficiently and recovers only truly associated covariate-taxon pairs.

## When to use

Use when fitting DMbvs_R or DTM regression to compositional microbiome counts (e.g. 100 subjects, 75 taxa, 30 covariates) and you must set an inclusion prior (e.g. prior='BB', a=1, b=9) to encode sparsity.

## When NOT to use

- When the compositional data have a known phylogenetic tree and branch-level (DTM) selection is the target rather than per-taxon DM selection
- When the covariate space is dense by design (few covariates all expected relevant) — a Beta-Binomial sparse prior tuned for 30 covariates over 75 taxa will over-shrink the signal

## Inputs

- Simulated or observed taxa count matrix (e.g. from simulate_DM with n_obs=100, n_taxa=75, n_vars=30, rho=0.3, seed=1)
- Covariate design matrix (30 simulated covariates)
- Inclusion prior specification (prior='BB' with a=1, b=9)
- MCMC settings: burn-in (1000 iterations), MPPI threshold (0.5)

## Outputs

- Posterior inclusion samples per covariate-taxon pair (MPPIs)
- Selected covariate-taxon pairs table (written to selected_pairs.csv)

## How to apply

Fit the Dirichlet-multinomial model with DMbvs_R on the simulated or real count data, specifying the inclusion prior parameterization — in the vignette benchmark, prior='BB' (Beta-Binomial) with hyperparameters a=1 and b=9 — so most of the 30 x 75 covariate-taxon associations are shrunk out. Run the MCMC discarding the first 1000 iterations as burn-in, then call selected_DM on the posterior samples with an MPPI threshold of 0.5 to extract selected covariate-taxon pairs. The rationale is that with B branches and P covariates there are 2^(B×P) potential models, so a sparse prior is needed to make model search tractable and to match the true signal (in the vignette simulation, 4 covariates truly associated with 4 taxa). Dependencies (Rcpp, RcppArmadillo, MCMCpack, mvtnorm, ggplot2, ape, igraph) must be installed before installing MicroBVS from GitHub.

## Related tools

- **MicroBVS** (R package providing simulate_DM, DMbvs_R, and selected_DM for Dirichlet-multinomial regression with Bayesian variable selection and configurable inclusion priors) — https://github.com/mkoslovsky/MicroBVS
- **Rcpp** (Dependency; the main DMbvs_R sampler functions operate in C++ via Rcpp and are wrapped by R functions) — https://github.com/mkoslovsky/MicroBVS
- **RcppArmadillo** (Dependency required for the C++ linear algebra underlying the MCMC sampler) — https://github.com/mkoslovsky/MicroBVS

## Examples

```
fit <- DMbvs_R(counts, X, prior='BB', a=1, b=9, burnin=1000); sel <- selected_DM(fit, cutoff=0.5); write.csv(sel, 'selected_pairs.csv')
```

## Evaluation signals

- Selected pairs at MPPI > 0.5 match the 4 covariate-taxon associations embedded by simulate_DM (ground truth)
- selected_pairs.csv exists and contains one row per selected covariate-taxon pair
- Convergence sanity: MCMC convergence 'is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model' — check that rho=0.3 correlation and prior sparsity do not stall the sampler
- The Beta-Binomial prior (a=1, b=9) yields sparse selections: only a small fraction of the 30 x 75 = 2250 candidate associations exceed the 0.5 threshold

## Limitations

- MCMC convergence depends on correlation structure between covariates and count data and on model sparsity; correlated covariates (rho=0.3 here) can degrade convergence
- No software version numbers, convergence diagnostics, or Zenodo/DOI archival record are provided for the package — only a GitHub home page (github.com/mkoslovsky/MicroBVS, MIT license)
- The Beta-Binomial hyperparameters (a=1, b=9) are vignette-specific; other parameterizations of covariates' prior inclusion probabilities are supported and may be preferable in other settings

## Evidence

- [full_text] In the vignette benchmark the Beta-Binomial inclusion prior is specified as prior='BB', a=1, b=9.: [evidence span withheld — anchor status: not_found]
- [results] For B branches and P covariates there are 2^(B×P) potential models, motivating a sparse inclusion prior.: "Specifically for B branches and P covariates, there are 2B×P potential models to choose from."
- [abstract] The underlying Bayesian model accommodates various parameterizations of covariates' prior probabilities of inclusion.: "The underlying Bayesian model accommodates phylogenetic structure in the abundance data and various parameterizations of covariates' prior probabilities of inclusion."
- [readme] The C++ sampler functions are wrapped by R functions and require dependencies installed before MicroBVS.: "The main functions operate in C++ via the R package Rcpp. These functions can be sourced by a set of wrapper functions that enable easy implementation of the code in the R environment."
- [results] Convergence depends on covariate correlation and model sparsity.: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
