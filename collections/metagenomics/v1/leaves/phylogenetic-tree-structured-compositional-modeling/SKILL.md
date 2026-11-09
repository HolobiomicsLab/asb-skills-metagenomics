---
name: phylogenetic-tree-structured-compositional-modeling
description: Use when the input is compositional taxa/count abundance data (e.g.,
  microbiome or diet-taxa tables) with a known phylogenetic tree, and the goal is
  to identify covariate-branch associations while respecting tree-like structure.
when_to_use_negative:
- The analysis target is a fixed, non-compositional response (e.g., a binary phenotype
  or continuous outcome) rather than covariate-branch selection over compositional
  taxa data
- The user expects a deterministic penalized-regression selection rather than Bayesian
  posterior inclusion — the paper contrasts a penalized approach that identified 271
  associations overall with the BVS method's 232 dietary factor-branch associations
- No MCMC budget or C++/Rcpp toolchain is available, since the core samplers operate
  in C++ via Rcpp wrappers and require burn-in and convergence checking
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_2269
tools:
- name: MicroBVS
  role: 'Primary R package providing DTM regression with Bayesian variable selection:
    simulate_DTM, DTMbvs_R fitting, and selected_DTM thresholding'
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Environment in which MicroBVS wrapper functions are run, with C++ backends
    via Rcpp
- name: Rcpp
  role: Dependency enabling the C++ MCMC samplers used by the MicroBVS wrapper functions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:61951ff983918881049e0b8d4d20c3afd7b8c2ab4fcd9e51ec1b391ae9c60392
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mkoslovsky/MicroBVS
  tools:
  - MicroBVS
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# phylogenetic-tree-structured-compositional-modeling

## Summary

Fit Dirichlet-tree multinomial (DTM) regression models with Bayesian variable selection to relate clinical covariates to phylogenetically structured compositional (e.g., microbiome) abundance data using the MicroBVS R package, selecting branch-by-covariate associations via MPPI thresholds.

## When to use

Use when the input is compositional taxa/count abundance data (e.g., microbiome or diet-taxa tables) with a known phylogenetic tree, and the goal is to identify covariate-branch associations while respecting tree-like structure.

## When NOT to use

- The analysis target is a fixed, non-compositional response (e.g., a binary phenotype or continuous outcome) rather than covariate-branch selection over compositional taxa data
- The user expects a deterministic penalized-regression selection rather than Bayesian posterior inclusion — the paper contrasts a penalized approach that identified 271 associations overall with the BVS method's 232 dietary factor-branch associations
- No MCMC budget or C++/Rcpp toolchain is available, since the core samplers operate in C++ via Rcpp wrappers and require burn-in and convergence checking

## Inputs

- Compositional taxa/branch abundance count table (e.g., DTM-simulated counts from simulate_DTM with 100 subjects, 50 covariates, 5-leaf tree, rho = 0.3, seed = 1)
- Phylogenetic tree relating taxa (e.g., 5-leaf ape phylo object)
- Subject covariate matrix (e.g., 50 covariates, possibly correlated, rho = 0.3)
- MCMC settings: burn-in iterations (e.g., 2500) and inclusion prior specification (e.g., Beta-Binomial with a = 0.1, b = 0.9)

## Outputs

- Set of selected branch-by-covariate associations at MPPI >= 0.50 (from selected_DTM)
- Posterior summaries/plots of inclusion probabilities and model fit produced by MicroBVS inference functions
- Recovery table of true positives vs. false positives (e.g., 9 of 15 planted associations recovered, zero false positives, written to dtmbvs_simulation_results.csv)

## How to apply

Install MicroBVS from GitHub (devtools::install_github('mkoslovsky/MicroBVS', build_vignettes = TRUE)) after installing its dependencies (Rcpp, RcppArmadillo, MCMCpack, mvtnorm, ggplot2, devtools, ape, igraph); core samplers run in C++ via Rcpp wrappers. Simulate or load DTM count data against the 5-leaf tree, then fit with DTMbvs_R, choosing a Beta-Binomial inclusion prior (e.g., a = 0.1, b = 0.9) that accommodates phylogenetic structure and covariate-specific prior inclusion probabilities. Discard the initial MCMC iterations as burn-in (2,500 in the vignette simulation) and apply selected_DTM with an MPPI threshold of 0.50 to call selected branch-by-covariate associations. Compare the selected set against planted/true associations to compute true and false positives; convergence depends on the correlation structure between covariates and count data and on model sparsity, so assess MCMC behavior accordingly.

## Related tools

- **MicroBVS** (Primary R package providing DTM regression with Bayesian variable selection: simulate_DTM, DTMbvs_R fitting, and selected_DTM thresholding) — https://github.com/mkoslovsky/MicroBVS
- **R** (Environment in which MicroBVS wrapper functions are run, with C++ backends via Rcpp)
- **Rcpp** (Dependency enabling the C++ MCMC samplers used by the MicroBVS wrapper functions)

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- On the vignette-style simulation (100 subjects, 50 covariates, 5-leaf tree, rho = 0.3, Beta-Binomial prior a = 0.1, b = 0.9, 2500-iteration burn-in), selection at MPPI >= 0.50 recovers 9 of 15 planted branch-by-covariate associations with zero false positives
- selected_DTM returns associations only for branch-covariate pairs whose marginal posterior probability of inclusion is >= 0.50
- The fitted model respects the 5-leaf tree structure: reported associations are at the branch level, not the individual-taxa level
- MCMC convergence is checked relative to covariate/count correlation structure and model sparsity, as convergence is reported to be highly dependent on both
- The package installs with build_vignettes = TRUE and all listed dependencies (Rcpp, RcppArmadillo, MCMCpack, mvtnorm, ggplot2, devtools, ape, igraph) resolve without error

## Limitations

- Convergence of the DTM MCMC algorithm is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model
- No software version numbers are provided for MicroBVS or its dependencies, and there is no DOI/Zenodo archival record — only the GitHub URL
- Case study data are referenced only via a package citation placeholder rather than a direct identifier, so exact input reproduction requires the vignette

## Evidence

- [abstract] MicroBVS R package for DTM-BVS: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection"
- [abstract] Model accommodates tree-like structure: "The underlying Bayesian model accommodates phylogenetic structure in the abundance data and various parameterizations of covariates' prior probabilities of inclusion."
- [results] Convergence caveat for MCMC: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
- [readme] C++ samplers via Rcpp wrappers: "The main functions operate in C++ via the R package Rcpp."
- [readme] Package installation from GitHub: "Then to install the MicroBVS package, simply run library(devtools) install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
