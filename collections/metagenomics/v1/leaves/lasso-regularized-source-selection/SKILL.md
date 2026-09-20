---
name: lasso-regularized-source-selection
description: Use when a sink community must be attributed to a small subset of truly
  contributing sources from a large panel of candidate source environments, and noncontributing-source
  noise must be suppressed.
when_to_use_negative:
- All candidate sources are already known to contribute, and the goal is only proportion
  estimation with no source-selection need — plain FEAST suffices.
- The input lacks the required metadata columns ('Env', 'SourceSink', 'id') or sample-to-source/sink
  mapping, since the model's sum-to-one, source-sink assignment cannot be constructed.
- No need to model an unknown/unobserved source exists and the source set is small
  — existing methods like SourceTracker2 may be adequate.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: EM algorithm with sparsity (L1-regularized least-squares source selection)
    that identifies contributing sources among a large candidate set
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Codebase STENSL branches from; supplies the EM source-tracking framework and
    Load_CountMatrix/Load_metadata input utilities
  repo: https://github.com/cozygene/FEAST
- name: R
  role: Implementation environment (R >= 3.4.4) in which STENSL/FEAST and its vignette
    run
  repo: https://github.com/cozygene/FEAST
- name: devtools
  role: Used to install FEAST/STENSL from GitHub
  repo: https://github.com/cozygene/FEAST
- name: Rcpp
  role: Dependency required prior to installing FEAST/STENSL
  repo: https://github.com/cozygene/FEAST
- name: RcppArmadillo
  role: Dependency required prior to installing FEAST/STENSL
  repo: https://github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:16febed65413f5db59729a793c642c1549c54f23e8191327e95e84809031a6d5
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
yes_rate: 0.5333333333333333
n_claims: 15
---

# lasso-regularized-source-selection

## Summary

Implements STENSL's sparsity-aware source-selection step: a least-squares fit of a sink microbial profile against candidate source profiles with L1 (Lasso) regularization on mixing proportions, using cross-validated lambda and EM to identify truly contributing sources among many candidates.

## When to use

Use when a sink community must be attributed to a small subset of truly contributing sources from a large panel of candidate source environments, and noncontributing-source noise must be suppressed.

## When NOT to use

- All candidate sources are already known to contribute, and the goal is only proportion estimation with no source-selection need — plain FEAST suffices.
- The input lacks the required metadata columns ('Env', 'SourceSink', 'id') or sample-to-source/sink mapping, since the model's sum-to-one, source-sink assignment cannot be constructed.
- No need to model an unknown/unobserved source exists and the source set is small — existing methods like SourceTracker2 may be adequate.

## Inputs

- m-by-n microbial count matrix (samples x taxa) with row names as sample IDs, e.g. a tab-delimited OTU table loaded via Load_CountMatrix
- m-by-3 metadata table with columns 'Env', 'SourceSink' ('Source'/'Sink'), and 'id', loaded via Load_metadata

## Outputs

- S1-by-S2 mixing-proportion matrix P (sinks x sources including an unknown source), each row summing to 1, NA where a source was unused for a sink (e.g. stensl_source_selection.csv)

## How to apply

Load the sink/source count table with Load_CountMatrix and metadata with Load_metadata (columns Env, SourceSink, id). Formulate source selection as a nonnegative least-squares problem between the sink profile x and the weighted source matrix Y^T*alpha under a sum-to-one constraint, enforcing sparsity via L1 regularization on alpha. For each fixed lambda, run expectation-maximization to infer parameters alpha and gamma; determine the optimal lambda by cross-validation over a grid (in experiments the authors used lambda = 1e-6), running a grid-search that returns the parameter set with the highest likelihood. Apply the sparse alpha as the mixing-proportion initialization for the unknown source, computing Y^{M+1} = max(0, x - alpha_hat^T * Y_argmax) by subtracting scaled counts of the top contributing source ranked by L1 regularization. Assemble results into an S1-by-S2 matrix P where each row sums to 1 and P[i,j] is the contribution of source j to sink i, writing NA where a source was not used for a sink.

## Related tools

- **STENSL** (EM algorithm with sparsity (L1-regularized least-squares source selection) that identifies contributing sources among a large candidate set) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Codebase STENSL branches from; supplies the EM source-tracking framework and Load_CountMatrix/Load_metadata input utilities) — https://github.com/cozygene/FEAST
- **R** (Implementation environment (R >= 3.4.4) in which STENSL/FEAST and its vignette run) — https://github.com/cozygene/FEAST
- **devtools** (Used to install FEAST/STENSL from GitHub) — https://github.com/cozygene/FEAST
- **Rcpp** (Dependency required prior to installing FEAST/STENSL) — https://github.com/cozygene/FEAST
- **RcppArmadillo** (Dependency required prior to installing FEAST/STENSL) — https://github.com/cozygene/FEAST

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "STENSL"); otus <- Load_CountMatrix(CountMatrix_path = "~/FEAST/Data_files/otu_example_multi.txt"); metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt")
```

## Evaluation signals

- Output file (e.g. stensl_source_selection.csv) exists and contains one row per sink.
- Every row of the mixing-proportion matrix P is nonnegative and sums to 1.
- The estimated alpha is sparse: most noncontributing candidate sources receive near-zero proportions, consistent with L1 regularization.
- The chosen lambda from the cross-validation/grid-search corresponds to the parameter set achieving the highest EM likelihood.
- NA entries appear only where a source was genuinely excluded from a sink's analysis, matching the P-matrix convention.

## Limitations

- The Lasso hyperparameter was fixed to a single value (lambda = 1e-6) in the paper's experiments without a reported sensitivity analysis.
- Detailed simulation and cross-validation protocols (e.g. Fig S3) are delegated to supplementary materials, not the main text.
- Accuracy claims rest on the paper's benchmarks; existing source-tracking methods add considerable error as the number of candidate sources grows, which motivated the sparsity approach but also means results depend on the candidate source set.

## Evidence

- [abstract] we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources: "we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources"
- [abstract] By incorporating sparsity into the estimation of potential source environments, STENSL improves the accuracy of true source contribution: "By incorporating sparsity into the estimation of potential source environments, STENSL improves the accuracy of true source contribution, while significantly reducing the noise introduced by"
- [discussion] For a fixed λ, we use expectation-maximization to infer parameters α and γ.: "For a fixed λ, we use expectation-maximization to infer parameters α and γ."
- [discussion] we subtract the scaled counts of the top contributing source, ranked based on the L-1 regularization and thus, Y^M+1=max(0,x-α^Yargmaxα^): [evidence span withheld — anchor status: not_found]
- [readme] *FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1.: "*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1. Pij is the contribution of source j"
- [methods] metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt"): "metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt")"
