---
name: dtm-mcmc-variable-selection
description: Use when you have genus-level OTU/ASV count data with a known phylogenetic
  tree and covariates (e.g., FFQ dietary variables) and need to identify which covariates
  associate with taxa branches via Bayesian variable selection in a DTM model.
when_to_use_negative:
- When the data are not compositional counts and/or no tree-like structure between
  taxa is available — although the package can handle data with or without a known
  tree, the branch-based selection logic of this skill presumes tree-structured DTM
  modeling
- When a penalized (non-Bayesian) sparse grouped-lasso DTM fit is desired instead,
  e.g. for AIC-based model selection over a lambda grid rather than posterior inclusion
  probabilities
- When the goal is phenotypic prediction or mediation analysis — the MicroBVS repository
  also contains separate joint outcome-prediction and compositional mediation models,
  which are distinct skills
tools:
- name: MicroBVS
  role: R package providing the DTM regression model with Bayesian variable selection,
    MCMC samplers, and posterior inference/plotting functions
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Environment in which the MicroBVS wrapper functions are run to fit the model
    and summarize posterior inclusion results
- name: Rcpp
  role: Interface through which the main MicroBVS functions operate in C++ during
    MCMC sampling
- name: RcppArmadillo
  role: C++ linear-algebra dependency required by the MicroBVS package installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:e11d43819bb6e8ddba138e88609bf4d7750c950726e1b0c3b3c2b1e3ac6aa575
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
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dtm-mcmc-variable-selection

## Summary

Perform Bayesian stochastic search variable selection for Dirichlet-tree multinomial (DTM) regression using the MicroBVS R package, selecting covariate-branch associations on compositional microbiome count data via marginal posterior probabilities of inclusion under a Beta-Binomial inclusion prior. It was used to identify dietary factor-branch associations in a 16S rRNA gut microbiome dietary case study.

## When to use

Use when you have genus-level OTU/ASV count data with a known phylogenetic tree and covariates (e.g., FFQ dietary variables) and need to identify which covariates associate with taxa branches via Bayesian variable selection in a DTM model.

## When NOT to use

- When the data are not compositional counts and/or no tree-like structure between taxa is available — although the package can handle data with or without a known tree, the branch-based selection logic of this skill presumes tree-structured DTM modeling
- When a penalized (non-Bayesian) sparse grouped-lasso DTM fit is desired instead, e.g. for AIC-based model selection over a lambda grid rather than posterior inclusion probabilities
- When the goal is phenotypic prediction or mediation analysis — the MicroBVS repository also contains separate joint outcome-prediction and compositional mediation models, which are distinct skills

## Inputs

- Genus-level OTU count matrix (compositional microbiome abundance data), e.g. 98 subjects in the case study
- Phylogenetic tree relating the taxa (branch structure for the Dirichlet-tree)
- Covariate matrix (e.g., 97 food-frequency-questionnaire dietary variables)
- Inclusion prior specification (Beta-Binomial a=1, b=1) and MCMC settings (iterations, burn-in)

## Outputs

- Posterior inclusion summaries per covariate-branch pair (marginal posterior probability of inclusion, MPPI)
- FDR-controlled list of selected covariate-branch associations (e.g., 232 associations at Bayesian FDR 0.01, MPPI >= 0.89)
- Result tables/plots from MicroBVS inference functions, e.g. dtm_benchmark_results.csv when benchmarking against the penalized approach

## How to apply

Load the genus-level OTU count matrix (98 subjects x 97 food-frequency-questionnaire dietary covariates in the benchmark case, shipped with the MicroBVS package) together with the phylogenetic tree, and fit the DTM regression in MicroBVS with a non-informative Beta-Binomial inclusion prior (a=1, b=1). Run 150,000 MCMC iterations with 75,000 burn-in; the C++-backed samplers are invoked through R wrapper functions. Select covariate-branch associations by controlling Bayesian FDR at 0.01, retaining associations with marginal posterior probability of inclusion (MPPI) >= 0.89. In the article's case study this yielded 232 dietary factor-branch associations, versus 271 for the Wang-Zhao penalized sparse grouped-lasso DTM (gamma=0.25, best model by minimum AIC), so treat the Bayesian-FDR/MPPI threshold as the key decision point and report both counts when benchmarking against the penalized alternative.

## Related tools

- **MicroBVS** (R package providing the DTM regression model with Bayesian variable selection, MCMC samplers, and posterior inference/plotting functions) — https://github.com/mkoslovsky/MicroBVS
- **R** (Environment in which the MicroBVS wrapper functions are run to fit the model and summarize posterior inclusion results)
- **Rcpp** (Interface through which the main MicroBVS functions operate in C++ during MCMC sampling)
- **RcppArmadillo** (C++ linear-algebra dependency required by the MicroBVS package installation)

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- Posterior summaries exist for every covariate-branch pair and the selected set satisfies both thresholds: Bayesian FDR controlled at 0.01 and MPPI >= 0.89
- MCMC run length matches the specified settings (150,000 iterations with 75,000 burn-in) and convergence diagnostics were inspected for the fitted model
- Association counts are in the expected range for this dataset: 232 associations for the Bayesian variable selection method vs 271 for the penalized approach on the same data
- Installation and run succeeded under the declared dependency set (Rcpp, RcppArmadillo, MCMCpack, mvtnorm, ggplot2, devtools, ape, igraph)
- Selections reproduce comparable dietary factors across taxa when compared with the penalized Wang-Zhao DTM fit, as the article observed for the penalized approach

## Limitations

- Convergence of the DTM MCMC is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model
- No software version numbers are provided for the MicroBVS package or its dependencies, and there is no changelog or archival (e.g., Zenodo) DOI for the code — only the GitHub URL
- Case study data availability is referenced only via a citation placeholder rather than a direct identifier, so exact data provenance must be resolved through the package's bundled dataset
- Note for adjacent MicroBVS code: as of 9/6/23 the joint outcome-prediction model corrected the update of the auxiliary parameters c_ij (Gibbs replaced by MH), which increases computation time — this fix applies to the joint model, not the DTM regression itself

## Evidence

- [abstract] Package purpose: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection"
- [abstract] Model accommodates tree structure and prior parameterizations: "The underlying Bayesian model accommodates phylogenetic structure in the abundance data and various parameterizations of covariates' prior probabilities of inclusion."
- [results] Case-study count from the Bayesian method: "We identified 232 dietary factor-branch associations with our Bayesian variable selection method for DTM regression models"
- [results] Convergence caveat: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
- [readme] C++-backed execution: "The main functions operate in C++ via the R package Rcpp. These functions can be sourced by a set of wrapper functions that enable easy implementation of the code in the R environment."
- [results] Comparison baseline from the penalized approach: "whereas the penalized approach identified 271 associations overall"
