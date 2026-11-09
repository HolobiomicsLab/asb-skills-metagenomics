---
name: mcmc-hyperparameter-and-burnin-tuning
description: Use when fitting a Bayesian DTM regression on compositional microbiome
  data (e.g. genus-level OTU counts) and you must choose inclusion-prior hyperparameters,
  MCMC run length/burn-in, and association-selection cutoffs before comparing variable-selection
  counts across methods.
when_to_use_negative:
- The data are not compositional counts with (or without) a tree structure to which
  a DTM model applies — e.g. they are already a feature table of normalized abundances
  rather than OTU counts.
- A deterministic penalized fit (e.g. sparse grouped-lasso DTM with lambda selected
  by AIC) is sufficient and MCMC sampling overhead is unnecessary.
- Convergence cannot be assessed because covariate correlation structure or model
  sparsity prevents the sampler from mixing within the feasible iteration budget.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MicroBVS
  role: R package providing the DTM regression with Bayesian variable selection, its
    MCMC sampler, and posterior inference/plotting functions
  repo: https://github.com/mkoslovsky/MicroBVS
- name: Rcpp
  role: Dependency enabling the C++ implementations of the main MCMC sampler functions
    used by MicroBVS wrappers
  repo: https://github.com/mkoslovsky/MicroBVS
- name: RcppArmadillo
  role: Dependency required to compile the C++ sampler code behind MicroBVS
  repo: https://github.com/mkoslovsky/MicroBVS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:7f2d77042d94eff142c6d459fffe380c6d01884000795780cafbe63f9b13942d
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# mcmc-hyperparameter-and-burnin-tuning

## Summary

Configure and tune the MCMC sampler of the MicroBVS Dirichlet-tree multinomial (DTM) regression with Bayesian variable selection, including the Beta-Binomial inclusion prior (a=1, b=1), iteration count (150,000 with 75,000 burn-in), and downstream Bayesian FDR/MPPI thresholds, so that posterior inclusion probabilities and association counts are stable and comparable.

## When to use

Use when fitting a Bayesian DTM regression on compositional microbiome data (e.g. genus-level OTU counts) and you must choose inclusion-prior hyperparameters, MCMC run length/burn-in, and association-selection cutoffs before comparing variable-selection counts across methods.

## When NOT to use

- The data are not compositional counts with (or without) a tree structure to which a DTM model applies — e.g. they are already a feature table of normalized abundances rather than OTU counts.
- A deterministic penalized fit (e.g. sparse grouped-lasso DTM with lambda selected by AIC) is sufficient and MCMC sampling overhead is unnecessary.
- Convergence cannot be assessed because covariate correlation structure or model sparsity prevents the sampler from mixing within the feasible iteration budget.

## Inputs

- Genus-level OTU count matrix (16S rRNA gut microbiome, 98 subjects)
- Food-frequency-questionnaire dietary covariate matrix (97 covariates)
- Phylogenetic tree structure over microbial taxa (Dirichlet-tree branches)

## Outputs

- Marginal posterior probabilities of inclusion (MPPI) per covariate-branch pair
- Set of dietary factor-branch associations passing Bayesian FDR 0.01 and MPPI >= 0.89 (e.g. 232 associations)
- Posterior summaries and convergence diagnostics for the DTM MCMC run

## How to apply

Install MicroBVS from GitHub via devtools after its C++ dependencies (Rcpp, RcppArmadillo) are present, since the main samplers are C++ implementations wrapped by R functions. Fit the DTM regression with a non-informative Beta-Binomial inclusion prior (a=1, b=1) on genus-level OTU counts versus dietary covariates (e.g. the package's 16S rRNA dataset: 98 subjects, 97 FFQ dietary covariates), running 150,000 MCMC iterations with 75,000 burn-in. Recognize that convergence is highly dependent on the correlation structure between covariates and count data as well as model sparsity, so verify sampler stability rather than assuming defaults suffice. Select covariate-branch associations by controlling Bayesian FDR at 0.01, retaining associations with marginal posterior probability of inclusion (MPPI) >= 0.89; these thresholds determine how many dietary factor-branch associations you report (232 in the case study). When benchmarking against penalized alternatives (Wang-Zhao sparse grouped-lasso DTM, gamma=0.25, AIC-selected lambda, 271 associations), keep the MCMC/prior configuration fixed so the comparison isolates the selection method rather than tuning choices.

## Related tools

- **MicroBVS** (R package providing the DTM regression with Bayesian variable selection, its MCMC sampler, and posterior inference/plotting functions) — https://github.com/mkoslovsky/MicroBVS
- **Rcpp** (Dependency enabling the C++ implementations of the main MCMC sampler functions used by MicroBVS wrappers) — https://github.com/mkoslovsky/MicroBVS
- **RcppArmadillo** (Dependency required to compile the C++ sampler code behind MicroBVS) — https://github.com/mkoslovsky/MicroBVS

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- The recorded MCMC configuration matches the benchmark settings: non-informative Beta-Binomial prior (a=1, b=1), 150,000 iterations, 75,000 burn-in.
- Association selection applies both cutoffs: Bayesian FDR controlled at 0.01 and MPPI >= 0.89; the reported association count (232 in the case study) is reproducible under these thresholds.
- A matched penalized run (Wang-Zhao sparse grouped-lasso, gamma=0.25, minimum-AIC lambda) on the same data yields a comparable-order count (271), serving as a cross-method sanity check.
- The MicroBVS package installs successfully via devtools after Rcpp/RcppArmadillo dependencies are installed, confirming the C++ sampler compiles.
- Convergence is explicitly checked/justified given covariate-correlation structure and model sparsity, since the article notes convergence is highly dependent on these factors.

## Limitations

- Convergence of the DTM MCMC is highly dependent on the correlation structure between covariates and count data, as well as the sparsity of the model.
- No software version numbers are provided for MicroBVS or its dependencies, and no changelog or archival DOI/Zenodo record exists for the code — only the GitHub URL.
- Case study data availability is referenced only via a citation placeholder rather than a direct dataset identifier.

## Evidence

- [results] convergence dependence: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
- [results] association count benchmark: "We identified 232 dietary factor-branch associations with our Bayesian variable selection method for DTM regression models"
- [results] penalized comparison count: "whereas the penalized approach identified 271 associations overall"
- [readme] package installation: "install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
- [discussion] dependency requirements: "Other requirements: R Rcpp RcppArmadillo ape MCMCpack mvtnorm ggplot2 GGMselect devtools ape igraph"
