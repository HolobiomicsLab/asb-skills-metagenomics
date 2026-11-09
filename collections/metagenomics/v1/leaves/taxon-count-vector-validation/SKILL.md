---
name: taxon-count-vector-validation
description: Use when a reconstructed source or sink taxon profile has been derived
  by arithmetic on count vectors (e.g., residual subtraction after L1 sparse source
  selection) and must be verified before downstream use.
when_to_use_negative:
- Input is already a validated source profile from FEAST's EM output (matrix P rows
  summing to 1) rather than a residual-derived count vector
- The mixing proportions are still unnormalized EM estimates and the L1 sparse selection
  step has not been run, so argmax(αhat) is not defined
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Produces the L1-regularized sparse selection output (x, Y, αhat) from which
    the residual unknown-source profile is reconstructed and validated
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: R
  role: Implementation environment for computing the clamped residual and writing/validating
    unknown_source_profile.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:06053ba05d2f0658e24597c08081f9dad19b230986bba6684e1ca87503dd540d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STENSL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# taxon-count-vector-validation

## Summary

Validates reconstructed taxon count vectors — specifically STENSL's unknown-source profile computed as the clamped residual Yhat_{M+1} = max(0, x − αhat*Y_{argmax αhat}) — by checking non-negativity, dimensional consistency with the sink vector, and correctness of the clamped subtraction.

## When to use

Use when a reconstructed source or sink taxon profile has been derived by arithmetic on count vectors (e.g., residual subtraction after L1 sparse source selection) and must be verified before downstream use.

## When NOT to use

- Input is already a validated source profile from FEAST's EM output (matrix P rows summing to 1) rather than a residual-derived count vector
- The mixing proportions are still unnormalized EM estimates and the L1 sparse selection step has not been run, so argmax(αhat) is not defined

## Inputs

- Sink taxon count vector x (n-taxa, from the sparse selection step of task_005)
- Source count matrix Y (sources × n taxa)
- Sparse mixing-proportion vector αhat from the L1-regularized (Lasso) selection step

## Outputs

- unknown_source_profile.csv containing the validated M+1-th (unknown) source profile vector

## How to apply

After STENSL's L1-regularized sparse selection produces the sink count vector x, source count matrix Y, and sparse mixing proportions αhat, identify the top contributing source as argmax(αhat) and compute the elementwise residual x − αhat*Y_{argmax αhat}, clamping negative entries to zero with max(0, .) since taxon counts cannot be negative. Write the resulting M+1-th (unknown) source profile to unknown_source_profile.csv. Validate three properties: every entry is non-negative, the vector has the same taxa dimension as x, and each entry equals the clamped residual of the subtraction formula. Rationale: the article's finding is that taxa not explained by contributing sources are taken as plausibly originating from the unknown source, so any dimension mismatch or negative value indicates an indexing or alignment error in the subtraction.

## Related tools

- **STENSL** (Produces the L1-regularized sparse selection output (x, Y, αhat) from which the residual unknown-source profile is reconstructed and validated) — https://github.com/cozygene/FEAST/tree/STENSL
- **R** (Implementation environment for computing the clamped residual and writing/validating unknown_source_profile.csv)

## Examples

```
R> unk <- pmax(0, x - max(alpha_hat) * Y[which.max(alpha_hat), ]); stopifnot(all(unk >= 0), length(unk) == length(x)); write.csv(unk, "unknown_source_profile.csv")
```

## Evaluation signals

- All entries of the output vector are ≥ 0 (clamping via max(0, .) confirmed)
- Output vector length equals the number of taxa (columns) in the sink count vector x
- Each entry equals max(0, x_i − αhat_max * Y_{argmax αhat, i}) when recomputed independently
- Output file unknown_source_profile.csv exists and parses as a single n-taxa numeric row
- Entries corresponding to taxa well explained by the top source are near zero; unexplained taxa retain positive residual mass

## Limitations

- The residual heuristic is an initialization, not a final estimate; the article notes the unknown source is subsequently refined within the EM framework
- The Lasso hyperparameter was fixed to λ = 1 × 10−6 without a reported sensitivity analysis, which affects which taxa are attributed to the unknown source
- Only the top-ranked source (argmax of αhat) is subtracted; contributions of smaller sources are not removed from the residual

## Evidence

- [discussion] we subtract the scaled counts of the top contributing source, ranked based on the L-1 regularization and thus, Y^M+1=max(0,x-α^Yargmaxα^): [evidence span withheld — anchor status: not_found]
- [full_text] taxa not explained by contributing sources are taken as plausibly originating from the unknown source: [evidence span withheld — anchor status: not_found]
- [discussion] formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [readme] An m by n count matrix, where m is the number samples and n is the number of taxa: "C - An m by n count matrix, where m is the number samples and n is the number of taxa."
