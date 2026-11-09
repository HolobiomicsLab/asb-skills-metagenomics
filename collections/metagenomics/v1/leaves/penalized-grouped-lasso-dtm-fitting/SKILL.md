---
name: penalized-grouped-lasso-dtm-fitting
description: Use when comparing a penalized (non-Bayesian) DTM approach against Bayesian
  variable selection on compositional count data with phylogenetic tree structure
  and many covariates (e.g. 97 FFQ dietary factors), to benchmark the number of detected
  taxon-covariate associations.
when_to_use_negative:
- When the analysis requires posterior inclusion probabilities (MPPI) or calibrated
  Bayesian FDR control per covariate-branch — use the MicroBVS Bayesian variable-selection
  DTM instead
- When the compositional data have no known tree-like structure, so branch-level grouped
  penalties are not meaningful
- When covariates are highly correlated and the model is dense, since sampler/fit
  convergence is highly dependent on the correlation structure between covariates
  and count data as well as model sparsity
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3697
tools:
- name: MicroBVS
  role: Source of the benchmark 16S rRNA dietary case-study dataset and the Bayesian
    DTM comparator whose association count (232) is benchmarked against the penalized
    fit
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Runtime environment in which the penalized DTM fit, the lambda/AIC grid search,
    and the comparison to the Bayesian results are executed
- name: Rcpp
  role: C++ backend through which the DTM main functions are run from R wrapper functions
- name: RcppArmadillo
  role: Dependency required in the R environment for the C++ DTM computations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:401fac04025d3a50622cbef3e9847c37d1ecd8ce7af477b8ab60894ed80e778b
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# penalized-grouped-lasso-dtm-fitting

## Summary

Fits the Wang and Zhao penalized Dirichlet-tree multinomial (DTM) regression with a sparse grouped-lasso penalty to genus-level microbiome compositional data, selecting covariate-branch associations without a Bayesian MCMC sampler. It serves as the frequentist comparator to the MicroBVS Bayesian variable-selection DTM on the same dietary gut-microbiome case study.

## When to use

Use when comparing a penalized (non-Bayesian) DTM approach against Bayesian variable selection on compositional count data with phylogenetic tree structure and many covariates (e.g. 97 FFQ dietary factors), to benchmark the number of detected taxon-covariate associations.

## When NOT to use

- When the analysis requires posterior inclusion probabilities (MPPI) or calibrated Bayesian FDR control per covariate-branch — use the MicroBVS Bayesian variable-selection DTM instead
- When the compositional data have no known tree-like structure, so branch-level grouped penalties are not meaningful
- When covariates are highly correlated and the model is dense, since sampler/fit convergence is highly dependent on the correlation structure between covariates and count data as well as model sparsity

## Inputs

- Genus-level OTU count matrix (16S rRNA gut microbiome benchmark data: 98 subjects x compositional taxa)
- Dietary covariate matrix of 97 food-frequency-questionnaire (FFQ) variables per subject
- Known phylogenetic tree structure relating the taxa (branches define the grouping used by the grouped lasso)

## Outputs

- List/table of selected dietary factor-branch associations from the AIC-optimal penalized model (271 associations overall in the case study)
- AIC values across the lambda grid identifying the best penalized model
- Comparison counts vs the Bayesian FDR-selected associations (232) for benchmarking

## How to apply

Load the genus-level OTU count dataset shipped with the MicroBVS R package (98 subjects, 97 food-frequency-questionnaire dietary covariates) and its known phylogenetic tree. Fit the Wang and Zhao penalized DTM regression with the sparse grouped-lasso penalty, fixing gamma = 0.25 and sweeping a grid of lambda values. Decision point: choose the final model by minimum AIC over the lambda grid rather than by posterior inclusion probabilities, since this frequentist approach has no MPPI or Bayesian FDR. Compile the selected dietary factor-branch associations and compare their count with the Bayesian pipeline, which controlled Bayesian FDR at 0.01 and retained associations with MPPI >= 0.89 (232 associations, vs 271 for the penalized fit). The main functions run in C++ via Rcpp inside R, so a working Rcpp/RcppArmadillo toolchain is required.

## Related tools

- **MicroBVS** (Source of the benchmark 16S rRNA dietary case-study dataset and the Bayesian DTM comparator whose association count (232) is benchmarked against the penalized fit) — https://github.com/mkoslovsky/MicroBVS
- **R** (Runtime environment in which the penalized DTM fit, the lambda/AIC grid search, and the comparison to the Bayesian results are executed)
- **Rcpp** (C++ backend through which the DTM main functions are run from R wrapper functions)
- **RcppArmadillo** (Dependency required in the R environment for the C++ DTM computations)

## Examples

```
library(devtools); install_github("mkoslovsky/MicroBVS", build_vignettes = TRUE)
```

## Evaluation signals

- Penalized fit on the benchmark dataset should identify 271 dietary factor-branch associations overall, versus 232 from the Bayesian FDR/MPPI pipeline (MPPI >= 0.89, Bayesian FDR 0.01)
- The selected model must be the minimum-AIC member of the lambda grid, with gamma fixed at 0.25 for the sparse grouped-lasso penalty
- Selected associations should tend to identify similar dietary factors across taxa, as observed in the article's penalized results
- Selected dietary factor-branch associations should be compiled into dtm_benchmark_results.csv alongside the Bayesian-selected associations
- R environment must satisfy the package requirements (Rcpp, RcppArmadillo, etc.) and the C++-backed main functions must run through the R wrapper functions

## Limitations

- Convergence of the DTM model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model
- The penalized approach tends to identify similar dietary factors across taxa, which can inflate the overall association count relative to Bayesian selection (271 vs 232)
- No software version numbers are provided for the tools and dependencies, and no changelog or archival (Zenodo) record exists for the code — only the GitHub URL
- The case study data are referenced only via citation '[15]' as data available in the R package, without a direct dataset identifier

## Evidence

- [results] Penalized vs Bayesian association counts: "We identified 232 dietary factor-branch associations with our Bayesian variable selection method for DTM regression models, whereas the penalized approach identified 271 associations overall"
- [results] Penalized consistency across taxa: "We observed that the penalized approach tended to identify similar dietary factors across taxa"
- [readme] C++/Rcpp implementation and R wrappers: "The main functions operate in C++ via the R package Rcpp. These functions can be sourced by a set of wrapper functions that enable easy implementation of the code in the R environment."
- [discussion] Required R dependencies: "Other requirements: R Rcpp RcppArmadillo ape MCMCpack mvtnorm ggplot2 GGMselect devtools ape igraph"
- [results] Convergence caveat: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model"
