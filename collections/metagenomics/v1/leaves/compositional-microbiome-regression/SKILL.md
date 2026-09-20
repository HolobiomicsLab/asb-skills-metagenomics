---
name: compositional-microbiome-regression
description: Use when regressing genus-level OTU/taxon counts (compositional data
  with or without a known phylogenetic tree) on covariates such as FFQ dietary variables,
  and you need Bayesian sparse variable selection with FDR control at the branch level.
when_to_use_negative:
- When the outcome is a phenotypic outcome requiring joint prediction rather than
  covariate-branch association screening — the README separates the joint AOAS model
  (with corrected MH updates for auxiliary parameters c_ij) from the DTM variable-selection
  package, so pick the corresponding model code instead
- When the data lack any compositional/taxa-count structure and are plain tabular
  features; the DTM likelihood and branch-based priors would be inappropriate
- When a penalized (frequentist grouped-lasso) DTM fit with AIC selection is specifically
  required rather than Bayesian posterior inference
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0091
tools:
- name: MicroBVS
  role: Primary R package implementing DTM regression with Bayesian variable selection;
    provides produce/summarize/plot functions for posterior inference
  repo: https://github.com/mkoslovsky/MicroBVS
- name: Rcpp
  role: Dependency enabling the C++ back-end of the main MCMC functions, sourced via
    R wrapper functions
  repo: https://github.com/mkoslovsky/MicroBVS
- name: RcppArmadillo
  role: Dependency required for the C++ linear-algebra routines behind the DTM samplers
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Environment in which the wrapper functions, vignettes, and inference/plotting
    tools are run
  repo: https://github.com/mkoslovsky/MicroBVS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:ef601c9b41ed526c2982002211c8f90eb7e55c1b22275e448dcb649b3390bb36
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
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# compositional-microbiome-regression

## Summary

Dirichlet-tree multinomial (DTM) regression with Bayesian variable selection, implemented in the MicroBVS R package, to identify covariate (e.g., dietary factor) associations with phylogenetically structured microbiome abundance data. It accommodates the tree-like structure of compositional taxa counts and reports marginal posterior probabilities of inclusion (MPPI) for each covariate-branch association.

## When to use

Use when regressing genus-level OTU/taxon counts (compositional data with or without a known phylogenetic tree) on covariates such as FFQ dietary variables, and you need Bayesian sparse variable selection with FDR control at the branch level.

## When NOT to use

- When the outcome is a phenotypic outcome requiring joint prediction rather than covariate-branch association screening — the README separates the joint AOAS model (with corrected MH updates for auxiliary parameters c_ij) from the DTM variable-selection package, so pick the corresponding model code instead
- When the data lack any compositional/taxa-count structure and are plain tabular features; the DTM likelihood and branch-based priors would be inappropriate
- When a penalized (frequentist grouped-lasso) DTM fit with AIC selection is specifically required rather than Bayesian posterior inference

## Inputs

- Genus-level OTU count (compositional) table from 16S rRNA data, e.g., 98 subjects x taxonomic branches
- Covariate matrix, e.g., 97 food-frequency-questionnaire dietary covariates
- Optional known phylogenetic tree structure over taxa (via the ape/igraph-supported graph priors)

## Outputs

- Posterior summaries of covariate-branch associations with MPPI values
- A filtered association list (e.g., 232 dietary factor-branch associations at Bayesian FDR 0.01 / MPPI >= 0.89), exportable as a CSV such as dtm_benchmark_results.csv
- Posterior inference and visualization objects produced by the package's summary/plot functions

## How to apply

Install MicroBVS from GitHub (devtools::install_github with build_vignettes = TRUE) after installing its dependencies (Rcpp, RcppArmadillo, MCMCpack, mvtnorm, ggplot2, devtools, ape, igraph); the heavy MCMC samplers run in C++ via Rcpp wrappers callable from R. Load the genus-level OTU count dataset (e.g., the benchmark 16S rRNA dataset of 98 subjects with 97 food-frequency-questionnaire dietary covariates) and fit the DTM regression with a non-informative Beta-Binomial inclusion prior (a=1, b=1), running 150,000 MCMC iterations with 75,000 burn-in. Select covariate-branch associations by controlling Bayesian FDR at 0.01, which corresponds to retaining associations with MPPI >= 0.89. Convergence depends on the correlation structure between covariates and count data and on model sparsity, so MCMC length should be assessed against those features. Associations from this Bayesian route can be benchmarked against penalized (sparse grouped-lasso, gamma=0.25, minimum-AIC) DTM alternatives on the same data.

## Related tools

- **MicroBVS** (Primary R package implementing DTM regression with Bayesian variable selection; provides produce/summarize/plot functions for posterior inference) — https://github.com/mkoslovsky/MicroBVS
- **Rcpp** (Dependency enabling the C++ back-end of the main MCMC functions, sourced via R wrapper functions) — https://github.com/mkoslovsky/MicroBVS
- **RcppArmadillo** (Dependency required for the C++ linear-algebra routines behind the DTM samplers) — https://github.com/mkoslovsky/MicroBVS
- **R** (Environment in which the wrapper functions, vignettes, and inference/plotting tools are run) — https://github.com/mkoslovsky/MicroBVS

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- Association counts should be reported at a stated Bayesian FDR threshold: the article's case study yields 232 dietary factor-branch associations at FDR 0.01 (MPPI >= 0.89), a useful sanity range when rerunning the same benchmark data
- MCMC settings should match the reported scheme (150,000 iterations, 75,000 burn-in) or be justified against covariate correlation and model sparsity since 'convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model'
- Package installation must succeed with all eight listed dependencies installed before devtools::install_github('mkoslovsky/MicroBVS', build_vignettes = TRUE)
- Cross-check against a penalized sparse grouped-lasso DTM (gamma=0.25, minimum-AIC lambda) which found 271 associations on the same data — large divergence from either count signals a pipeline error
- Output should include per-branch MPPI values so the FDR-0.01/MPPI>=0.89 filter can be reproduced and audited

## Limitations

- No software version numbers are provided for MicroBVS or its dependencies (R, Rcpp, RcppArmadillo, ape, MCMCpack, mvtnorm, ggplot2, GGMselect, devtools, igraph)
- MCMC convergence is highly dependent on the correlation structure between covariates and count data and on model sparsity; convergence diagnostics and prior version numbers are not reported in the results section
- Case study data availability is only referenced via citation placeholder '[15]' rather than a direct identifier, and there is no DOI/Zenodo archival record for the code — only the GitHub URL
- The penalized approach tended to identify similar dietary factors across taxa, and model-space size grows as 2^(B x P) for B branches and P covariates, so exhaustive comparison is infeasible

## Evidence

- [abstract] We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection: "We present MicroBVS, an R package for Dirichlet-tree multinomial models with Bayesian variable selection"
- [results] Bayesian FDR case study result: "We identified 232 dietary factor-branch associations with our Bayesian variable selection method for DTM regression models"
- [results] Convergence dependence: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
- [discussion] Package dependencies: "Other requirements: R Rcpp RcppArmadillo ape MCMCpack mvtnorm ggplot2 GGMselect devtools ape igraph"
- [readme] README installation instructions: "Then to install the MicroBVS package, simply run 

library(devtools)
install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
