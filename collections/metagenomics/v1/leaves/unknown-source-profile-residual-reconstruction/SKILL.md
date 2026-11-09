---
name: unknown-source-profile-residual-reconstruction
description: Use when a STENSL L1-regularized sparse selection output (sink vector
  x, source matrix Y, sparse alphahat) is available and an explicit profile for the
  unknown/unobserved source must be derived, not just its mixing proportion.
when_to_use_negative:
- You only need the unknown source's mixing proportion, which STENSL's EM/FEAST-style
  output P already reports as an 'unknown' column — not its taxon-level profile
- No sparse source selection has been run yet (x, Y, alphahat are unavailable), since
  the residual heuristic is defined on the L1 solution
- Candidate source set contains no plausible contributing source, making argmax(alphahat)
  uninformative for subtraction
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Provides the sparsity-incorporating expectation-maximization and L1-regularized
    least-squares source selection whose output (alphahat) defines the residual subtraction
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: R
  role: Implementation language for computing the clamped residual and writing unknown_source_profile.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:d16798a5e6e755e02dce02906deee74c3389a6d577557ed8757b41362e131ed1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STENSL
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# unknown-source-profile-residual-reconstruction

## Summary

Reconstructs the unknown (unobserved) source profile in STENSL by taking the clamped residual of the sink counts minus the top contributing source's counts scaled by its L1-regularized mixing proportion: Yhat_{M+1} = max(0, x − alphahat * Y_{argmax alphahat}). This leverages STENSL's sparse selection step so taxa unexplained by contributing sources become the estimated unknown-source profile.

## When to use

Use when a STENSL L1-regularized sparse selection output (sink vector x, source matrix Y, sparse alphahat) is available and an explicit profile for the unknown/unobserved source must be derived, not just its mixing proportion.

## When NOT to use

- You only need the unknown source's mixing proportion, which STENSL's EM/FEAST-style output P already reports as an 'unknown' column — not its taxon-level profile
- No sparse source selection has been run yet (x, Y, alphahat are unavailable), since the residual heuristic is defined on the L1 solution
- Candidate source set contains no plausible contributing source, making argmax(alphahat) uninformative for subtraction

## Inputs

- Sink count vector x (taxa-dimension count vector)
- Source count matrix Y (sources × taxa)
- Sparse mixing-proportion vector alphahat from STENSL's L1-regularized (Lasso) source selection step

## Outputs

- unknown_source_profile.csv — the M+1-th (unknown) source profile vector, same taxa dimension as x, non-negative clamped residuals

## How to apply

Load the sparse selection output comprising the sink count vector x, the source count matrix Y, and the sparse mixing-proportion vector alphahat produced by STENSL's Lasso/L1-regularized least-squares step (formulated between x and YTα). Identify the top contributing source as argmax(alphahat), the source ranked highest by the L1 regularization. Compute the elementwise residual x − alphahat * Y_{argmax alphahat}, scaling the top source's counts by alphahat before subtraction, and clamp negative entries to zero with max(0, ·) since taxon counts cannot be negative. Write the resulting vector as the M+1-th (unknown) source profile. Rationale: taxa accurately reconstructed by candidate sources are explained by them, so residual taxa are plausibly attributable to the unknown source.

## Related tools

- **STENSL** (Provides the sparsity-incorporating expectation-maximization and L1-regularized least-squares source selection whose output (alphahat) defines the residual subtraction) — https://github.com/cozygene/FEAST/tree/STENSL
- **R** (Implementation language for computing the clamped residual and writing unknown_source_profile.csv)

## Examples

```
unknown_profile <- pmax(0, x - alpha_hat[max(which(alpha_hat == max(alpha_hat))), ] * Y[which.max(alpha_hat), ]); write.csv(unknown_profile, "unknown_source_profile.csv")
```

## Evaluation signals

- Output vector is non-negative everywhere (max(0, ·) clamping applied)
- Output vector has the same taxa dimension as the sink vector x
- Each entry equals the clamped residual x_i − alphahat_max * Y[argmax alphahat, i]
- argmax(alphahat) corresponds to the source ranked highest by the L1 regularization, verified against the sparse selection output
- Taxa with high sink counts but near-zero counts in the top source should dominate the reconstructed unknown profile

## Limitations

- The reconstruction is a heuristic residual from only the single top-ranked contributing source; taxa partially explained by other contributing sources may leak into the unknown profile
- The Lasso hyperparameter was fixed at λ = 1 × 10−6 without a reported sensitivity analysis, which may affect the alphahat ranking driving the subtraction
- Detailed simulation/evaluation protocols are delegated to supplementary Text S1, so no in-context accuracy benchmark is available for this specific step

## Evidence

- [discussion] Unknown source initialization by subtracting top contributing source counts: "we subtract the scaled counts of the top contributing source, ranked based on the L-1 regularization and thus, Y^M+1=max(0,x-α^Yargmaxα^)"
- [discussion] Lasso-based sparse source selection: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [abstract] Sparsity rationale: "By incorporating sparsity into the estimation of potential source environments, STENSL improves the accuracy of true source contribution"
- [abstract] Unknown source identification capability: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
