---
name: sparse-mixing-proportion-interpretation
description: Use when an L1-regularized least-squares mixture fit over candidate source
  environments has produced a sparse alpha-hat and the unobserved (unknown) source
  contribution to a sink must be reconstructed or the top contributing source identified.
when_to_use_negative:
- When the goal is only mixing-proportion estimation with FEAST's expectation-maximization
  output (matrix P); this skill reconstructs the unknown-source profile from the sparse
  L1 solution instead.
- When the candidate source set already includes the true origin of all sink taxa,
  the residual unknown-source profile will be near zero and adds no information.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Provides the L1-regularized sparse source-selection step whose alpha-hat is
    interpreted and whose top-ranked source drives the unknown-source residual reconstruction
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Codebase in which STENSL is implemented; supplies the count matrix and metadata
    loading utilities (Load_CountMatrix, Load_metadata)
  repo: https://github.com/cozygene/FEAST
- name: R
  role: Implementation language for loading inputs, computing the clamped residual
    formula, and writing unknown_source_profile.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:adcc4f98e55b534b59e6f7ad7f109328ac8056a0d2aa9961fbed4471c2a0bb5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - STENSL
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sparse-mixing-proportion-interpretation

## Summary

Interprets the L1-regularized sparse mixing-proportion vector (alpha-hat) produced by STENSL's Lasso source-selection step, using the top-ranked source to reconstruct the unknown-source profile via the residual heuristic Y^M+1 = max(0, x - alpha-hat * Y_argmax).

## When to use

Use when an L1-regularized least-squares mixture fit over candidate source environments has produced a sparse alpha-hat and the unobserved (unknown) source contribution to a sink must be reconstructed or the top contributing source identified.

## When NOT to use

- When the goal is only mixing-proportion estimation with FEAST's expectation-maximization output (matrix P); this skill reconstructs the unknown-source profile from the sparse L1 solution instead.
- When the candidate source set already includes the true origin of all sink taxa, the residual unknown-source profile will be near zero and adds no information.

## Inputs

- Sink count vector x (taxa-dimension counts)
- Source count matrix Y (sources x taxa)
- Sparse mixing-proportion vector alpha-hat from the L1-regularized (Lasso) source-selection step

## Outputs

- Unknown-source profile vector Y^M+1 (clamped residual counts, same taxa dimension as x)
- Index/identity of the top contributing source (argmax of alpha-hat)

## How to apply

Take the sink count vector x, the source count matrix Y, and the sparse mixing-proportion vector alpha-hat emitted by STENSL's Lasso-based quadratic program (least squares between x and Y^T alpha with L1 regularization). Identify the top contributing source as argmax(alpha-hat), since it is ranked by the L1 regularization. Compute the elementwise residual x - alpha-hat * Y_{argmax(alpha-hat)}, scaling the top source's counts by alpha-hat before subtracting from the sink, then clamp negative entries to zero with max(0, .) because taxon counts cannot be negative. Write the resulting M+1-th (unknown) source profile vector for downstream use. This exploits the fact that taxa not explained by contributing sources are plausibly of unknown origin.

## Related tools

- **STENSL** (Provides the L1-regularized sparse source-selection step whose alpha-hat is interpreted and whose top-ranked source drives the unknown-source residual reconstruction) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Codebase in which STENSL is implemented; supplies the count matrix and metadata loading utilities (Load_CountMatrix, Load_metadata)) — https://github.com/cozygene/FEAST
- **R** (Implementation language for loading inputs, computing the clamped residual formula, and writing unknown_source_profile.csv)

## Examples

```
unknown_profile <- pmax(x - alpha_hat[which.max(alpha_hat)] * Y[which.max(alpha_hat), ], 0)
```

## Evaluation signals

- Output unknown-source vector is non-negative everywhere (max(0, .) clamping applied correctly)
- Output vector has the same taxa dimension as the sink count vector x
- Each entry of the output equals x_i - alpha-hat * Y_{argmax, i} for entries where the residual is positive, and 0 otherwise
- The argmax of alpha-hat is a source that received non-zero weight in the sparse L1 solution
- In FEAST's S1 by S2 output matrix P, rows sum to 1 and the unknown column is present as the M+1-th source

## Limitations

- The Lasso hyperparameter lambda was fixed to 1e-6 in the authors' experiments without a reported sensitivity analysis, which may affect which source is ranked top.
- The residual heuristic attributes any sink taxa unexplained by the top-ranked contributing source to the unknown source; mis-ranking of the top source propagates error into the reconstructed profile.
- Detailed simulation and validation protocols are in supplementary Text S1 and are not included in the main document.

## Evidence

- [discussion] Unknown source initialization by subtracting top contributing source counts: [evidence span withheld — anchor status: not_found]
- [discussion] Lasso-based sparse source selection: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [readme] FEAST output includes unknown source: "*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source)."
