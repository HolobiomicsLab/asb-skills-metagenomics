---
name: convex-combination-mixing-proportion-estimation
description: Use when a sink microbial community must be decomposed into contributions
  from many candidate source environments (hundreds), and sparsity is needed to identify
  the truly contributing sources while discounting noncontributing ones.
when_to_use_negative:
- When the candidate source set is small and manually curated — plain FEAST already
  quantifies proportions of different microbial samples in a target community without
  needing sparsity-based selection
- When the input is not a compositional count table with paired metadata distinguishing
  Source vs Sink rows (required column names 'Env', 'SourceSink', 'id' must be respected)
- When the goal is identifying which sources contribute but no unknown/unobserved
  source is possible — STENSL's unknown-source initialization assumes unobserved sources
  may exist
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Expectation-maximization algorithm with sparsity that identifies contributing
    sources among many candidate environments and estimates sparse mixing proportions
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Base codebase and R package whose FEAST() function returns the S1-by-S2 mixing-proportion
    matrix P; STENSL is a branch of this codebase
  repo: https://github.com/cozygene/FEAST
- name: R
  role: Implementation language (>= 3.4.4) for loading data and running the EM/Lasso
    source-selection fit
  repo: https://github.com/cozygene/FEAST
- name: devtools
  role: Used to install FEAST from GitHub
  repo: https://github.com/cozygene/FEAST
- name: Rcpp
  role: Dependency required before installing FEAST
- name: RcppArmadillo
  role: Dependency required before installing FEAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:15d6e78e8fa22d33a6652b2a9f047250c0d752fc7f404ec2f6e6390b1b292ed4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - Rcpp
  - RcppArmadillo
  - STENSL
  - devtools
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# convex-combination-mixing-proportion-estimation

## Summary

Estimates microbial source contributions to a sink community by modeling the sink as a convex combination of contributing sources, nuisance sources, and an unknown source, using expectation-maximization with L1-regularized (Lasso) sparse source selection as implemented in STENSL, a branch of the FEAST codebase.

## When to use

Use when a sink microbial community must be decomposed into contributions from many candidate source environments (hundreds), and sparsity is needed to identify the truly contributing sources while discounting noncontributing ones.

## When NOT to use

- When the candidate source set is small and manually curated — plain FEAST already quantifies proportions of different microbial samples in a target community without needing sparsity-based selection
- When the input is not a compositional count table with paired metadata distinguishing Source vs Sink rows (required column names 'Env', 'SourceSink', 'id' must be respected)
- When the goal is identifying which sources contribute but no unknown/unobserved source is possible — STENSL's unknown-source initialization assumes unobserved sources may exist

## Inputs

- m-by-n count matrix (samples x taxa) loaded via Load_CountMatrix
- m-by-3 metadata table with columns 'Env', 'SourceSink', 'id' loaded via Load_metadata
- sink/source grouping (different_sources_flag specifying whether each sink is tested with a distinct source group)

## Outputs

- S1-by-S2 mixing-proportion matrix P (rows sum to 1; P[i,j] = contribution of source j to sink i; NA = source not used for that sink), including an unknown-source column
- Saved output file, e.g. 'demo_FEAST.txt' (or stensl_source_selection.csv in the reconstructed workflow)

## How to apply

Load the sink/source count matrix with Load_CountMatrix and the matching metadata (columns 'Env', 'SourceSink', 'id') with Load_metadata. Formulate source selection as a least-squares fit between the sink profile x and Y^T*alpha with a sum-to-one constraint, enforcing sparsity via L1 regularization on alpha; determine the Lasso hyperparameter lambda by cross-validation across a candidate grid (in the paper's experiments lambda = 1e-6 was used), running expectation-maximization to infer alpha and gamma for each fixed lambda and a grid-search returning the parameter set with the highest likelihood. Initialize the unknown source as Y^{M+1} = max(0, x - alpha_hat^T * Y_argmax) by subtracting the scaled counts of the top contributing source ranked by L1 regularization. Return an S1-by-S2 matrix P where each row sums to 1 and P[i,j] is the contribution of source j to sink i (NA indicates a source was not used for that sink); in the README FEAST saves this as '<outfile>_FEAST.txt'.

## Related tools

- **STENSL** (Expectation-maximization algorithm with sparsity that identifies contributing sources among many candidate environments and estimates sparse mixing proportions) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Base codebase and R package whose FEAST() function returns the S1-by-S2 mixing-proportion matrix P; STENSL is a branch of this codebase) — https://github.com/cozygene/FEAST
- **R** (Implementation language (>= 3.4.4) for loading data and running the EM/Lasso source-selection fit) — https://github.com/cozygene/FEAST
- **devtools** (Used to install FEAST from GitHub) — https://github.com/cozygene/FEAST
- **Rcpp** (Dependency required before installing FEAST)
- **RcppArmadillo** (Dependency required before installing FEAST)

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "STENSL"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile = "demo")
```

## Evaluation signals

- Every row of the output matrix P is nonnegative and sums to 1
- Output has one row per sink and one column per source plus an unknown source (S1-by-S2 matrix P)
- NA entries appear only where a source was genuinely not used in the analysis of a given sink
- Selected lambda (from cross-validation grid-search) yields the highest-likelihood EM parameter set (alpha, gamma)
- Estimated mixing proportions are sparse: noncontributing sources receive near-zero weight rather than diffuse error, as expected from L1 regularization

## Limitations

- In the paper's experiments the Lasso hyperparameter was fixed at lambda = 1e-6 without a reported sensitivity analysis
- Figure S3 (cross-validation detail) is only available as an external download, not embedded in the article
- Detailed simulation protocols and in vitro assembly protocols are delegated to supplementary Texts S1 and S2 and are not in the main document
- No software version for STENSL/FEAST is specified; FEAST/STENSL install license is NOASSERTION
- Existing source tracking methods (motivating STENSL) add considerable error when hundreds of potential sources are considered — mixing proportions from such methods should not be trusted in that regime

## Evidence

- [abstract] STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
- [discussion] Lasso source selection and EM with lambda cross-validation: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [discussion] For a fixed λ, we use expectation-maximization to infer parameters α and γ.: "For a fixed λ, we use expectation-maximization to infer parameters α and γ."
- [discussion] Unknown source initialization: [evidence span withheld — anchor status: not_found]
- [discussion] STENSL code is a branch of FEAST: "The code for STENSL is a branch of the FEAST codebase, which can be found on https://github.com/cozygene/FEAST/tree/STENSL"
- [readme] README output format for mixing-proportion matrix P: "*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."
