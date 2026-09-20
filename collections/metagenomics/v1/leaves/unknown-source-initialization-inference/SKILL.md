---
name: unknown-source-initialization-inference
description: Use when a sink community cannot be fully explained by candidate sources
  — i.e., residual taxa remain after sparse Lasso-based source selection, and the
  fraction of an unobserved source environment must be estimated.
when_to_use_negative:
- When all sink mass is fully explained by candidate sources and no unknown/unobserved
  environment is suspected — the unknown column is unnecessary.
- When the analysis goal is ordinary FEAST-style proportion estimation over a small,
  fixed source set without source exploration or unknown-source estimation.
- When input is raw sequencing reads rather than a taxa-by-sample count table with
  matching Env/SourceSink/id metadata.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: EM algorithm with sparsity whose unknown-source initialization and L1-regularized
    source selection this skill implements; distributed as a branch of the FEAST codebase
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Parent codebase providing Load_metadata, Load_CountMatrix, and the FEAST output
    convention (S1-by-S2 matrix P with rows summing to 1, unknown column, NA for unused
    sources)
  repo: https://github.com/cozygene/FEAST
- name: Rcpp
  role: Compiled dependency required to install and run the FEAST/STENSL R package
  repo: https://github.com/cozygene/FEAST
- name: RcppArmadillo
  role: Linear-algebra backend used by FEAST/STENSL for the EM and least-squares computations
  repo: https://github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:3cee57fc456b17a637069c3943a6b87cd3e414fd54c5e8d907ebf3f97a524676
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# unknown-source-initialization-inference

## Summary

Initializes the unobserved (unknown) source in STENSL microbial source tracking by subtracting the scaled counts of the top contributing source, ranked by L1 regularization, so that unexplained sink taxa can be attributed to an unsequenced environment.

## When to use

Use when a sink community cannot be fully explained by candidate sources — i.e., residual taxa remain after sparse Lasso-based source selection, and the fraction of an unobserved source environment must be estimated.

## When NOT to use

- When all sink mass is fully explained by candidate sources and no unknown/unobserved environment is suspected — the unknown column is unnecessary.
- When the analysis goal is ordinary FEAST-style proportion estimation over a small, fixed source set without source exploration or unknown-source estimation.
- When input is raw sequencing reads rather than a taxa-by-sample count table with matching Env/SourceSink/id metadata.

## Inputs

- Sink microbial count profile x (taxa counts for a sink sample)
- Source count matrix Y (candidate source environments x taxa)
- Sparse mixing proportions alpha_hat from L1-regularized least-squares source selection
- Metadata table with columns 'Env', 'SourceSink', 'id' (loaded via Load_metadata)
- Count matrix loaded via Load_CountMatrix

## Outputs

- Initialized unknown source profile Y^{M+1} = max(0, x - alpha_hat^T * Y_argmax) (nonnegative taxa-count vector)
- S1-by-S2 mixing-proportion matrix P (rows sum to 1; P[i,j] = contribution of source j to sink i, including the unknown source column; NA where a source was not used), e.g. saved as stensl_source_selection.csv

## How to apply

After formulating source selection as a nonnegative least-squares fit with L1 regularization between the sink profile x and Y^T*alpha, obtain the sparse mixing proportions alpha_hat via EM for fixed lambda and select lambda by cross-validation across a grid (Fig S3), returning the parameter set with the highest likelihood. Then initialize the unknown source as Y^{M+1} = max(0, x - alpha_hat^T * Y_argmax) by subtracting the scaled counts of the top contributing source ranked by L1 regularization; max(0,·) truncates negative residuals so the unknown-source profile stays nonnegative. This initialized unknown source enters the STENSL EM mixture model alongside candidate sources, so that unexplained sink mass is assigned to the unknown column of the output matrix. In the returned S1-by-S2 matrix P, each row sums to 1, P[i,j] is the contribution of source j to sink i, and NA marks sources not used for a sink.

## Related tools

- **STENSL** (EM algorithm with sparsity whose unknown-source initialization and L1-regularized source selection this skill implements; distributed as a branch of the FEAST codebase) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Parent codebase providing Load_metadata, Load_CountMatrix, and the FEAST output convention (S1-by-S2 matrix P with rows summing to 1, unknown column, NA for unused sources)) — https://github.com/cozygene/FEAST
- **Rcpp** (Compiled dependency required to install and run the FEAST/STENSL R package) — https://github.com/cozygene/FEAST
- **RcppArmadillo** (Linear-algebra backend used by FEAST/STENSL for the EM and least-squares computations) — https://github.com/cozygene/FEAST

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "STENSL"); otus <- Load_CountMatrix(CountMatrix_path="~/FEAST/Data_files/otu_example_multi.txt"); metadata <- Load_metadata(metadata_path="~/FEAST/Data_files/metadata_example_multi.txt")
```

## Evaluation signals

- Unknown-source profile is nonnegative: every element of Y^{M+1} = max(0, x - alpha_hat^T * Y_argmax) is >= 0 after truncation.
- Output matrix P has one row per sink and every row is nonnegative and sums to 1.
- Sources not used in the analysis of a sink are marked NA in P rather than given a spurious zero contribution.
- Lambda chosen by cross-validation corresponds to the highest-likelihood parameter set returned by the grid-search over the lambda range.
- Output file (e.g., stensl_source_selection.csv or demo_FEAST.txt) exists and follows the FEAST output schema with an 'unknown' column.

## Limitations

- In the reported experiments lambda was fixed to 1e-6 without a reported sensitivity analysis, so cross-validation behavior near this default is not characterized.
- Figure S3 (cross-validation) and detailed simulation/assembly protocols are only in supplementary materials, limiting reproducibility of hyperparameter tuning details.
- Accuracy of prior source-selection strategies is very low in the presence of an unknown source, so results depend heavily on the quality of the L1-regularized initialization.

## Evidence

- [discussion] Unknown source initialization by subtracting top contributing source counts: [evidence span withheld — anchor status: not_found]
- [discussion] Cross-validation and grid-search for lambda: [evidence span withheld — anchor status: not_found]
- [discussion] EM inference of alpha and gamma for fixed lambda: "For a fixed λ, we use expectation-maximization to infer parameters α and γ."
- [readme] FEAST output matrix convention including unknown source: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."
- [abstract] Sparsity enables identification of contributing sources: "an expectation-maximization algorithm with sparsity that enables the identification of contributing sources among a large set of potential microbial environments"
