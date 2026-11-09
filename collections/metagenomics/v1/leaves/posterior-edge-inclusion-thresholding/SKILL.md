---
name: posterior-edge-inclusion-thresholding
description: Use when you have MCMC posterior samples of inclusion indicators from
  a MicroBVS DM/DTM Bayesian variable selection run (e.g. prior='MRF_unknown' or Beta-Binomial
  a=0.1, b=0.9) and must convert them into a discrete selected graph or association
  set.
when_to_use_negative:
- The compositional count data have no tree structure and no MRF/graph-based prior
  was fit, so no posterior inclusion indicators over edges or branches exist to threshold
- You have raw abundance or feature tables rather than fitted MicroBVS posterior samples
  — run DMbvs_R/DTMbvs_R first
- A penalized frequentist selection approach was used instead of Bayesian variable
  selection; inclusion-probability thresholding does not apply to those outputs
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MicroBVS
  role: Provides selected_DM (with G=TRUE for graphical selection) and selected_DTM
    wrapper functions that threshold posterior inclusion probabilities into adjacency
    matrices and selected association sets
  repo: https://github.com/mkoslovsky/MicroBVS
- name: R
  role: Runtime environment for running MicroBVS samplers and applying the selection/thresholding
    functions
  repo: https://github.com/mkoslovsky/MicroBVS
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03640-0
    title: 'MicroBVS: Dirichlet-tree multinomial regression models with Bayesian variable
      selection - an R package'
  source_content_hash: sha256:2206af6598e57d16d3f8088cd58d017a6f0b72b5036f75f9cbacaef48895eaa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mkoslovsky/MicroBVS
  tools:
  - MicroBVS
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# posterior-edge-inclusion-thresholding

## Summary

Extract a learned graphical structure (or selected branch-by-covariate associations) from MicroBVS posterior samples by thresholding marginal posterior probabilities of inclusion — edge inclusions via selected_DM(G=TRUE) for MRF priors, or MPPI >= 0.50 via selected_DTM for DTM regression models.

## When to use

Use when you have MCMC posterior samples of inclusion indicators from a MicroBVS DM/DTM Bayesian variable selection run (e.g. prior='MRF_unknown' or Beta-Binomial a=0.1, b=0.9) and must convert them into a discrete selected graph or association set.

## When NOT to use

- The compositional count data have no tree structure and no MRF/graph-based prior was fit, so no posterior inclusion indicators over edges or branches exist to threshold
- You have raw abundance or feature tables rather than fitted MicroBVS posterior samples — run DMbvs_R/DTMbvs_R first
- A penalized frequentist selection approach was used instead of Bayesian variable selection; inclusion-probability thresholding does not apply to those outputs

## Inputs

- Posterior samples of edge-inclusion probabilities from DMbvs_R with prior='MRF_unknown' (after discarding burn-in iterations)
- Posterior samples of branch-by-covariate inclusion probabilities from DTMbvs_R (after burn-in, e.g. 2500 iterations)
- MPPI threshold (e.g. 0.50) and a logical flag G=TRUE for graphical selection in selected_DM
- Optionally, the true simulated adjacency matrix or planted association set for recovery evaluation

## Outputs

- Binary square symmetric adjacency matrix over covariates (learned MRF graph)
- List of selected covariate-branch terms / branch-by-covariate associations at the MPPI threshold
- Selection summary table (e.g. true positives and false positives), exportable as CSV such as mrf_adjacency.csv or dtmbvs_simulation_results.csv

## How to apply

After running DMbvs_R or DTMbvs_R and discarding burn-in (e.g. 1000 iterations for the MRF_unknown run, 2500 for the DTM vignette simulation), feed the posterior samples of edge-inclusion or branch-by-covariate inclusion probabilities into the selection wrapper: selected_DM with G=TRUE for the graphical (MRF) case, or selected_DTM with an MPPI threshold of 0.50 for the DTM case. Thresholding is the decision point that turns continuous posterior probabilities into a binary adjacency matrix or a finite association set; for MRF_unknown, the thresholded edge-inclusion probabilities yield the adjacency matrix over covariates, which you validate as square, symmetric, and binary. Export the selected adjacency and covariate terms (e.g. to mrf_adjacency.csv) or compare the selected associations against the planted truth to count true/false positives (e.g. 9 of 15 true associations, 0 false positives). Rationale: MicroBVS operates over 2^(B×P) potential branch-by-covariate models, so posterior-inclusion thresholding is the package's intended model-selection step rather than ad-hoc coefficient screening.

## Related tools

- **MicroBVS** (Provides selected_DM (with G=TRUE for graphical selection) and selected_DTM wrapper functions that threshold posterior inclusion probabilities into adjacency matrices and selected association sets) — https://github.com/mkoslovsky/MicroBVS
- **R** (Runtime environment for running MicroBVS samplers and applying the selection/thresholding functions) — https://github.com/mkoslovsky/MicroBVS

## Examples

```
sel <- selected_DM(post_samples, G = TRUE)
```

## Evaluation signals

- The recovered adjacency matrix is square, symmetric, and contains binary (0/1) edge indicators over the simulated covariates
- The exported CSV (e.g. mrf_adjacency.csv) exists and parses as a valid CSV
- For the DTM vignette simulation (100 subjects, 50 covariates, 5-leaf tree, rho=0.3, Beta-Binomial a=0.1/b=0.9, MPPI >= 0.50), selection recovers 9 of 15 planted branch-by-covariate associations with zero false positives
- The MRF_unknown model on block-structured covariance data (sigma_ij = 0.7) identifies the strong within-block edges, with partial recovery (fewer edges) at sigma_ij = 0.3 and 0.5
- The selection step follows an explicit burn-in discard (1000 iterations for MRF_unknown; 2500 for DTMbvs_R) before thresholding

## Limitations

- Recovery is partial at weaker covariance correlations: the MRF_unknown model selected fewer edges when sigma_ij = 0.3 or 0.5, so thresholding can miss true weak edges
- Convergence of the model is highly dependent on the correlation structure between covariates and count data, as well as the sparsity level of the model
- No software version numbers are provided for the MicroBVS package or its dependencies, and the joint model's revised MH update increases computation time

## Evidence

- [results] model space: "Specifically for B branches and P covariates, there are 2B×P potential models to choose from."
- [abstract] prior parameterizations: "The underlying Bayesian model accommodates phylogenetic structure in the abundance data and various parameterizations of covariates' prior probabilities of inclusion."
- [readme] installation: "install_github( "mkoslovsky/MicroBVS", build_vignettes = TRUE)"
- [readme] package scope: "Various functions are available that produce, summarize, and plot the results for inference."
