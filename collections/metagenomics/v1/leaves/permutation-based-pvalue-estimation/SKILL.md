---
name: permutation-based-pvalue-estimation
description: Use when a microbiome association test statistic (e.g. SPU/aSPU/AMAT
  T_AMAT) has no tractable analytic null distribution and an empirical P-value must
  be calibrated by permuting the phenotype vector.
when_to_use_negative:
- A valid asymptotic or kernel-based analytic null distribution exists for the test
  statistic (e.g. kernel-matrix-based tests with closed-form nulls), making permutation
  an unnecessary computational burden.
- The analysis is not testing an association statistic at all (e.g. pure OTU table
  QC or taxonomic classification), where permutation calibration is out of scope.
- Extremely low B is required for runtime reasons; a truncated null distribution cannot
  support P-values at the 0.05 threshold.
edam_operation: http://edamontology.org/operation_3665
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AMAT
  role: R package implementing the AMAT test whose permutation-based P-value estimation
    this skill calibrates; shares null statistics between PSPU(γ) and the final P-value.
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which the permutation loop and empirical P-value
    calculation are executed.
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:3cf89aab38ad03f90b761bc43992eae703e1b24df32ca6702e03baba339e2eae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/kzb193/AMAT
  repo_url: https://github.com/kzb193/AMAT
  tools:
  - AMAT
  - LDM
  - MiHC
  - OMiAT
  - OMiRKAT
  - R
  - aMiSPU
  edam_operation: http://edamontology.org/operation_3665
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# permutation-based-pvalue-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate empirical P-values for microbiome multivariate association tests by permuting the outcome vector and comparing the observed statistic against the null distribution; in AMAT the same null statistics are reused to build both the PSPU(γ) combined statistic and the final P-value.

## When to use

Use when a microbiome association test statistic (e.g. SPU/aSPU/AMAT T_AMAT) has no tractable analytic null distribution and an empirical P-value must be calibrated by permuting the phenotype vector.

## When NOT to use

- A valid asymptotic or kernel-based analytic null distribution exists for the test statistic (e.g. kernel-matrix-based tests with closed-form nulls), making permutation an unnecessary computational burden.
- The analysis is not testing an association statistic at all (e.g. pure OTU table QC or taxonomic classification), where permutation calibration is out of scope.
- Extremely low B is required for runtime reasons; a truncated null distribution cannot support P-values at the 0.05 threshold.

## Inputs

- OTU abundance matrix compositional-transformed and OTU-wise standardized (Z*)
- Adjusted response vector (continuous or binary phenotype)
- Selected feature set S from the upstream screening step (Algorithm 1)
- Number of permutations B (500 for AMAT testing; 10,000 for method comparisons)

## Outputs

- Empirical P-value P_AMAT as a single numeric value in [0,1] (e.g. written to pamat_pvalue.csv)
- Vector of B null permutation test statistics
- Empirical power and type I error estimates at the 0.05 level across scenarios (for benchmarking use)

## How to apply

Generate B permutations of the adjusted response vector (B = 500 in the AMAT testing stage; B = 10,000 was used for the paper's power/type-I-error comparisons of AMAT, aMiSPU, LDM, MiHC, OMiAT and OMiRKAT). For each permuted response, recompute the test statistic — including re-running feature screening and the adaptive test on the permuted data — so the same set of null statistics serves both the PSPU(γ) combination step and the final P-value, greatly reducing permutation cost. Estimate the empirical P-value as P = (1/B) Σ I[T(b) ≤ T_obs] and report a single numeric value in [0,1]. Reusing null statistics across the SPU path and the final calibration is the key efficiency decision: if the screening step is stochastic, it must be applied inside each permutation to keep the null valid.

## Related tools

- **AMAT** (R package implementing the AMAT test whose permutation-based P-value estimation this skill calibrates; shares null statistics between PSPU(γ) and the final P-value.) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which the permutation loop and empirical P-value calculation are executed.)

## Evaluation signals

- The output P-value is a single numeric value in [0,1].
- The same set of B null permutation statistics is reused for the PSPU(γ) statistic and the final P-value (verify by inspecting the permutation loop).
- P_AMAT = (1/B) Σ I[T(b) ≤ T_AMAT] computed over the full B = 500 (or 10,000) null statistics.
- Type I error under permuted (null) phenotypes is controlled near 0.05 at the 0.05 significance level.
- The screening step is re-applied inside each permutation, not fixed to the observed-data selection.

## Limitations

- Permutation cost scales with B and with re-running feature screening per permutation; AMAT mitigates this by reusing null statistics for both PSPU(γ) and the final P-value.
- Empirical P-values are bounded by the resolution of B: with B = 500 the smallest attainable P-value is coarser than with B = 10,000.
- MiHC permutation-based P-values were conservative (especially with binary outcomes), so permutation calibration alone does not guarantee correct type I error for every test.

## Evidence

- [other] In AMAT, permutations are used to compute both the inherent SPU P-values and the final P-value, and the computational cost is greatly reduced: [evidence span withheld — anchor status: not_found]
