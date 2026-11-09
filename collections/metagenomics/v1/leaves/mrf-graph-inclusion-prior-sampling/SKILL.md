---
name: mrf-graph-inclusion-prior-sampling
description: Use when compositional count data (e.g. taxa abundances) must be related
  to covariates that are believed to be correlated (block covariance structure), and
  the dependency graph among covariates is not known a priori and must be inferred
  jointly with variable selection.
when_to_use_negative:
- When the graphical structure among covariates is known a priori — use a graph-based
  inclusion prior with a fixed graph rather than MRF_unknown
- When covariates are approximately independent with no meaningful correlation structure
  to model
- When the data are not compositional (e.g. raw feature tables without a multinomial/Dirichlet-tree
  structure) or no tree-like structure over the compositional outcome is available
  or desired
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: MicroBVS
  role: R package providing the DMbvs_R sampler (prior='MRF_unknown'), simulate_DM
    data generation, and selected_DM inference for edge-inclusion and covariate-term
    selection
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Programming environment in which MicroBVS wrapper functions are executed;
    core functions operate in C++ via Rcpp
- name: Rcpp
  role: Dependency enabling the C++-backed sampler functions of MicroBVS to run from
    R
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:e930a9db08da842ce3344115b06e22e2cf174376ad8285b51001749afa429fd1
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mrf-graph-inclusion-prior-sampling

## Summary

Fits Dirichlet-multinomial (DM) Bayesian variable selection with a Markov random field (MRF) inclusion prior in which the graphical structure among covariates is unknown, sampling both edge inclusions and covariate-branch terms and recovering the learned adjacency matrix. Implemented via the MicroBVS R package (DMbvs_R sampler with prior='MRF_unknown' and selected_DM with G=TRUE).

## When to use

Use when compositional count data (e.g. taxa abundances) must be related to covariates that are believed to be correlated (block covariance structure), and the dependency graph among covariates is not known a priori and must be inferred jointly with variable selection.

## When NOT to use

- When the graphical structure among covariates is known a priori — use a graph-based inclusion prior with a fixed graph rather than MRF_unknown
- When covariates are approximately independent with no meaningful correlation structure to model
- When the data are not compositional (e.g. raw feature tables without a multinomial/Dirichlet-tree structure) or no tree-like structure over the compositional outcome is available or desired

## Inputs

- Simulated or observed compositional count data (taxa abundance counts) with associated covariates
- Covariates with correlated (block covariance) structure, e.g. within-block correlations of 0.7, 0.5, and 0.3
- MCMC settings: burn-in iterations (1000 discarded) and number of posterior samples

## Outputs

- Learned adjacency matrix over the covariates (square, symmetric, binary edge indicators)
- List of selected covariate-branch terms from thresholded posterior inclusions
- CSV file mrf_adjacency.csv containing the learned adjacency matrix and selected terms
- Posterior edge-inclusion probabilities from the MRF_unknown sampler

## How to apply

Construct a block-structured covariance matrix Sigma with within-block correlations (e.g. 0.7, 0.5, 0.3) and generate covariates plus compositional count data using MicroBVS's simulate_DM function. Run the DMbvs_R sampler with prior='MRF_unknown', discarding the first 1000 iterations as burn-in, to obtain posterior samples of edge inclusions and covariate terms; the MRF prior lets edge inclusion probabilities inform covariate term inclusion since correlated covariates tend to be included together. Apply selected_DM with G=TRUE to the posterior samples and threshold the posterior edge-inclusion probabilities to extract the learned adjacency matrix among covariates. Then extract the selected covariate-branch terms from the thresholded posterior inclusions and write the adjacency matrix and terms to mrf_adjacency.csv. Interpret recovery against the true graph: stronger within-block correlations (sigma_ij = 0.7) are identified more reliably, while weaker ones (0.3, 0.5) yield fewer selected edges and only partial graph recovery.

## Related tools

- **MicroBVS** (R package providing the DMbvs_R sampler (prior='MRF_unknown'), simulate_DM data generation, and selected_DM inference for edge-inclusion and covariate-term selection) — https://github.com/mkoslovsky/MicroBVS
- **R** (Programming environment in which MicroBVS wrapper functions are executed; core functions operate in C++ via Rcpp)
- **Rcpp** (Dependency enabling the C++-backed sampler functions of MicroBVS to run from R)

## Examples

```
fit <- DMbvs_R(Y, X, prior='MRF_unknown', burnin=1000); sel <- selected_DM(fit, G=TRUE); write.csv(list(adjacency=sel$G, terms=sel$terms), 'mrf_adjacency.csv')
```

## Evaluation signals

- Output file mrf_adjacency.csv exists and is a valid CSV
- Recovered adjacency matrix is square and symmetric over the simulated covariates with binary edge indicators
- Edge selection tracks correlation strength: stronger within-block relations (sigma_ij = 0.7) recovered, fewer edges selected at sigma_ij = 0.3 and 0.5
- First 1000 iterations discarded as burn-in before collecting posterior samples
- Selected covariate-branch terms correspond to the thresholded posterior inclusions returned by selected_DM with G=TRUE

## Limitations

- Only partially recovers the true graph when within-block correlations are weak (sigma_ij = 0.3 and 0.5), selecting fewer edges than truly present
- Convergence of the DTM MCMC algorithm is highly dependent on the correlation structure between covariates and count data as well as model sparsity; no version numbers or convergence diagnostics reported
- Requires multiple R dependencies (Rcpp, RcppArmadillo, ape, MCMCpack, mvtnorm, ggplot2, GGMselect, devtools, igraph) installed before the package can be used

## Evidence

- [results] Graph-based inclusion priors: "the MicroBVS package can be specified with graph-based inclusion priors, similar to [26, 27]. See the vignette for details"
- [results] MRF_unknown recovery of block structure: [evidence span withheld — anchor status: not_found]
- [other] Sampler and data generation functions: [evidence span withheld — anchor status: not_found]
- [other] Thresholded edge-inclusion extraction: [evidence span withheld — anchor status: not_found]
- [readme] Package installation and dependencies: "install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
- [results] Convergence dependence: "convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model."
