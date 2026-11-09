---
name: cross-validated-hyperparameter-grid-search
description: Use when fitting a regularized (L1/Lasso) mixture model — e.g., STENSL
  source selection with sink profile x vs. Y^T*α — and the sparsity level must be
  chosen from data rather than fixed arbitrarily.
when_to_use_negative:
- When the number of candidate sources is small and full (non-sparse) source tracking
  is adequate — the paper notes existing methods like FEAST/SourceTracker2 are sufficient
  in low-source-count settings, and STENSL's sparsity targets large candidate sets.
- When no hyperparameter is tunable or the model has already been fit with a validated
  fixed λ (e.g., the authors' λ = 1 × 10−6) — a grid search would add cost without
  benefit.
- When the input is already a mixing-proportion output (S1 × S2 matrix P); the grid
  search operates on the count matrix and metadata, not on downstream contribution
  tables.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Expectation-maximization algorithm with sparsity whose Lasso hyperparameter
    λ is tuned by the cross-validated grid search
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Codebase STENSL branches from; provides Load_CountMatrix/Load_metadata loaders
    and the EM source-tracking framework in which the λ grid-search runs
  repo: https://github.com/cozygene/FEAST
- name: RcppArmadillo
  role: Compiled linear-algebra dependency for the R implementation of the EM/quadratic-program
    steps
  repo: https://github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:594f999021600fdb0237a5cb2e6c040b9fa7794b7e0e1b92d3d76c24ea919798
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-validated-hyperparameter-grid-search

## Summary

Determines an optimal Lasso regularization hyperparameter λ for sparse microbial source tracking (STENSL) by cross-validation over a candidate grid, then running expectation-maximization at each fixed λ and returning the α, γ parameter set with the highest likelihood.

## When to use

Use when fitting a regularized (L1/Lasso) mixture model — e.g., STENSL source selection with sink profile x vs. Y^T*α — and the sparsity level must be chosen from data rather than fixed arbitrarily.

## When NOT to use

- When the number of candidate sources is small and full (non-sparse) source tracking is adequate — the paper notes existing methods like FEAST/SourceTracker2 are sufficient in low-source-count settings, and STENSL's sparsity targets large candidate sets.
- When no hyperparameter is tunable or the model has already been fit with a validated fixed λ (e.g., the authors' λ = 1 × 10−6) — a grid search would add cost without benefit.
- When the input is already a mixing-proportion output (S1 × S2 matrix P); the grid search operates on the count matrix and metadata, not on downstream contribution tables.

## Inputs

- Sink/source taxa count matrix (m samples × n taxa, tab-delimited, loaded via Load_CountMatrix)
- Metadata table with 'Env', 'SourceSink', 'id' columns (loaded via Load_metadata)
- Grid of candidate Lasso hyperparameter λ values

## Outputs

- Optimal λ selected by cross-validation (highest-likelihood parameter set)
- Sparse mixing-proportion vector α per sink (many entries at/near zero)
- S1 × S2 matrix P of source contributions to sinks (rows sum to 1, NA where a source was unused), e.g. saved as stensl_source_selection.csv

## How to apply

Formulate source selection as a nonnegative least-squares fit between the sink profile x and Y^T*α under a sum-to-one constraint, enforcing sparsity via L1 regularization on α. Define a grid of candidate λ values and, for each fixed λ, run expectation-maximization to infer the mixing proportions α and the unknown-source parameter γ. Use cross-validation across the grid (as in Fig S3 of the STENSL paper) to score candidate λ values, then run the grid-search across the λ range and return the parameter set with the highest likelihood. Note the authors themselves used λ = 1 × 10−6 in their experiments, so a sensitivity analysis or CV curve is advisable when applying the grid search to new data. The selected sparse α then serves as the mixing-proportion initialization for downstream unknown-source estimation (Y^{M+1} = max(0, x − α̂^T * Y_argmax)).

## Related tools

- **STENSL** (Expectation-maximization algorithm with sparsity whose Lasso hyperparameter λ is tuned by the cross-validated grid search) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Codebase STENSL branches from; provides Load_CountMatrix/Load_metadata loaders and the EM source-tracking framework in which the λ grid-search runs) — https://github.com/cozygene/FEAST
- **RcppArmadillo** (Compiled linear-algebra dependency for the R implementation of the EM/quadratic-program steps) — https://github.com/cozygene/FEAST

## Examples

```
metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt"); otus <- Load_CountMatrix(CountMatrix_path = "~/FEAST/Data_files/otu_example_multi.txt"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- Cross-validation likelihood curve over the λ grid should show a clear optimum (as in Fig S3); the selected λ is the grid value maximizing held-out likelihood.
- Returned α from the highest-likelihood parameter set must be sparse: most candidate sources receive near-zero mixing proportion when hundreds of environments are considered.
- Resulting S1 × S2 matrix P must have every row nonnegative and summing to 1, with NA entries only where a source was not used for that sink.
- Output file (e.g., stensl_source_selection.csv) must have exactly one row per sink and columns per source plus an unknown column.
- Number of contributing sources identified should not grow with the size of the candidate source pool (STENSL's stated advantage over FEAST/SourceTracker2).

## Limitations

- The paper fixes λ = 1 × 10−6 in its experiments without a reported sensitivity analysis; the cross-validation figure (Fig S3) is only available as an external download, so users must reproduce the CV sweep themselves.
- Detailed simulation protocol for validating λ selection is delegated to supplementary Text S1 and is not included in the main document.
- No software version is specified for STENSL/FEAST, complicating reproducibility of grid-search results.

## Evidence

- [discussion] For a fixed λ, we use expectation-maximization to infer parameters α and γ.: "For a fixed λ, we use expectation-maximization to infer parameters α and γ."
- [discussion] Grid-search selects highest-likelihood parameters: "a grid-search algorithm to perform EM across a range of values, returning the set of parameters which obtains the highest likelihood"
- [discussion] Cross-validation determines lambda: "we determine an optimal value for hyperparameter λ through cross-validation"
- [discussion] Lasso regularization of least-squares source fit: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [readme] FEAST output is S1 by S2 matrix P with rows summing to 1: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."
- [discussion] Fixed lambda value in experiments: "In our experiments, we chose λ = 1 × 10−6"
