---
name: compositional-microbiome-data-simulation
description: Use when benchmarking a variable-selection or mediation-selection method
  for compositional count data and you need simulated data with known truth (e.g.,
  which taxa, covariates, or balances are truly active) to score sensitivity/specificity
  of the selection.
when_to_use_negative:
- You need inference on real observational microbiome data with unknown truth — simulation
  provides synthetic benchmarks, not biological findings
- Your model is not a Dirichlet(-tree) multinomial or joint DM-linear/mediation model
  of compositional counts; the embedded ground truth of these simulators is tailored
  to those MicroBVS model classes
- You need phylogenetically structured trees or compositional data generated under
  mechanisms outside the vignette's default simulation settings
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0625
tools:
- name: MicroBVS
  role: Supplies the simulate_DM, Simulate_MedDM, and simulate_DMLM simulation functions
    that generate compositional count data with known embedded active features for
    benchmarking DTM variable selection and mediation selection
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Environment in which the simulation functions are called and the simulated
    counts are passed to the MCMC fitting routines
- name: RcppArmadillo
  role: Backend dependency (with Rcpp) enabling the C++ code that powers the MicroBVS
    package's simulation and MCMC functions
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:38cbf3490466629e3dcea225f29a8a1b460f4197e28970590f22c1d44f53f5cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mkoslovsky/MicroBVS
  tools:
  - BVSMed
  - MicroBVS
  - R
  - Rcpp
  - RcppArmadillo
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# compositional-microbiome-data-simulation

## Summary

Generate synthetic compositional microbiome count data with embedded ground truth (active taxa, covariate-taxon associations, active balances, mediator taxa) using MicroBVS's simulate_DM, Simulate_MedDM, and simulate_DMLM functions, providing controlled benchmarks for evaluating Bayesian variable selection in Dirichlet-tree multinomial (DTM) regression and mediation models.

## When to use

Use when benchmarking a variable-selection or mediation-selection method for compositional count data and you need simulated data with known truth (e.g., which taxa, covariates, or balances are truly active) to score sensitivity/specificity of the selection.

## When NOT to use

- You need inference on real observational microbiome data with unknown truth — simulation provides synthetic benchmarks, not biological findings
- Your model is not a Dirichlet(-tree) multinomial or joint DM-linear/mediation model of compositional counts; the embedded ground truth of these simulators is tailored to those MicroBVS model classes
- You need phylogenetically structured trees or compositional data generated under mechanisms outside the vignette's default simulation settings

## Inputs

- Simulation configuration: number of observations, number of taxa, number of covariates (or exposure/mediator layout for mediation), and correlation rho between covariates
- Random seed fixing the embedded ground truth (e.g., seed=1 for simulate_DM, seed=123 for Simulate_MedDM)
- Specification of the true active set: covariate-taxon associations, first-N balances, or active mediator taxon indices

## Outputs

- Compositional microbiome count matrix (e.g., 100 x 20 or 100 x 75 taxa counts) with associated covariates/exposure and outcome
- Ground-truth labels: indices of active mediators, covariate-taxon associations, or active covariates/balances for validation
- Recovery comparison artifacts (e.g., selected_pairs.csv, selection_med2_results.csv, selected_DMLM_results.csv)

## How to apply

Choose the simulator matching the model under test: simulate_DM for DM regression with covariates (e.g., n_obs=100, n_taxa=75, n_vars=30, rho=0.3, seed=1, embedding 4 known covariate-taxon associations), Simulate_MedDM for mediation (e.g., n_obs=100, n_taxa=20, seed=123, with taxa 3, 4, and 5 as the true relative mediators), or simulate_DMLM for joint DM-linear models (e.g., 50 subjects, 50 taxa, 50 covariates, 10 active covariates with the first 10 balances active). Set the seed so results are reproducible and the ground-truth active set is fixed. Fit the downstream model (e.g., DMbvs_R with prior='BB', a=1, b=9; MCMC_Med; dm_lm_bvs_R with 10,000 thinned-by-10 iterations) on the simulated counts, then apply the corresponding selection routine (selected_DM, Selection_Med1/2/3, or selected_DMLM) with MPPI threshold 0.5. Score the selections against the simulation truth to judge recovery of the embedded active features.

## Related tools

- **MicroBVS** (Supplies the simulate_DM, Simulate_MedDM, and simulate_DMLM simulation functions that generate compositional count data with known embedded active features for benchmarking DTM variable selection and mediation selection) — https://github.com/mkoslovsky/MicroBVS
- **R** (Environment in which the simulation functions are called and the simulated counts are passed to the MCMC fitting routines)
- **RcppArmadillo** (Backend dependency (with Rcpp) enabling the C++ code that powers the MicroBVS package's simulation and MCMC functions)

## Examples

```
library(MicroBVS); sim <- Simulate_MedDM(n_obs=100, n_taxa=20, seed=123)
```

## Evaluation signals

- simulate_DM with n_obs=100, n_taxa=75, n_vars=30, seed=1 must embed exactly 4 covariate-taxon associations as the recoverable ground truth
- Simulate_MedDM with n_obs=100, n_taxa=20, seed=123 must treat taxa 3, 4, and 5 as the true relative mediators (e.g., Strategy 3 recovers all three; Strategy 1 recovers only 3rd and 4th; Strategy 2 recovers only the 3rd)
- simulate_DMLM with 10 active covariates must activate the first 10 balances, so selected_xi/selected_zeta can be checked against these indices
- Selections extracted with MPPI threshold 0.5 (selected_DM, Selection_Med, selected_DMLM with burn-in, e.g., 1000 or 500) should be compared field-by-field against the embedded truth records
- Fixed seeds (e.g., seed=1 or 123) must make the simulated data and truth labels reproducible across runs

## Limitations

- Convergence of the DTM MCMC algorithms 'is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model', so simulated-data recovery can vary with rho and active-set size
- No software version numbers are provided for MicroBVS or its dependencies, and there is no changelog or Zenodo/DOI archival record for the code, complicating exact reproduction of simulated results
- Default simulation settings (e.g., active mediators 1-3 at taxa 3-5) are illustrative vignette configurations; selection strategies may miss true mediators (Strategy 2 misses taxa 4 and 5), so a single strategy's failure is not evidence the simulation is wrong

## Evidence

- [abstract] Abstract framing of the package that hosts the simulators: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection"
- [results] Package purpose and simulation/benchmark context in results: "the MicroBVS package can be specified with graph-based inclusion priors, similar to [26, 27]. See the vignette for details"
- [readme] Installation instructions grounding the R/Rcpp execution environment: "The main functions operate in C++ via the R package Rcpp. These functions can be sourced by a set of wrapper functions that enable easy implementation of the code in the R environment."
- [readme] README pointing to vignettes for running each model, which contain the simulation workflows: [evidence span withheld — anchor status: external_doc]
