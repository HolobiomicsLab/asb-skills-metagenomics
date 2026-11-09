---
name: generalized-linear-model-association-testing
description: Use when testing association between a microbial taxa-set (e.g. ≥5 taxa
  at genus/family/order/class/phylum level) and a continuous or binary outcome, where
  most taxa in the clade are noise and a data-adaptive multivariate GLM test with
  permutation calibration is needed.
when_to_use_negative:
- The input is a single taxon or a taxa-set with fewer than five taxa — the article
  restricts association analysis to taxa-sets containing at least five taxa
- The association signal is extremely sparse — AMAT was not the most powerful method
  in that simulation regime
- The outcome requires a link function outside the standard GLM framework (linear/logistic)
  implemented by the AMAT package
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AMAT
  role: 'R package implementing the two-stage adaptive association test: distance-correlation
    feature screening followed by aSPU testing with tuning parameters Γ = {2,3,4,8}
    and permutation P-value calibration'
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which AMAT and the competing multivariate association
    tests (aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) are executed
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:b0a0039518a6f04ce1f89c1475a48e1259f3492d311ee810a9cf49b293a961ce
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# generalized-linear-model-association-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs multivariate microbiome association testing of H0: β = 0 between a selected set of taxa and a continuous or binary outcome using AMAT's data-adaptive sum of powered score (aSPU/SPU) statistic within a generalized linear model framework, with permutation-based P-value calibration. Feature selection via distance correlation learning shrinks noise taxa to exactly zero before testing.

## When to use

Use when testing association between a microbial taxa-set (e.g. ≥5 taxa at genus/family/order/class/phylum level) and a continuous or binary outcome, where most taxa in the clade are noise and a data-adaptive multivariate GLM test with permutation calibration is needed.

## When NOT to use

- The input is a single taxon or a taxa-set with fewer than five taxa — the article restricts association analysis to taxa-sets containing at least five taxa
- The association signal is extremely sparse — AMAT was not the most powerful method in that simulation regime
- The outcome requires a link function outside the standard GLM framework (linear/logistic) implemented by the AMAT package

## Inputs

- OTU abundance count matrix (e.g. 856 OTUs × 60 samples from the throat microbiome) after QC and filtering
- Adjusted response vector (continuous or binary outcome, e.g. smoking status)
- Selected feature set S from the upstream Algorithm 1 distance-correlation screening step
- Permutation count B (500) and tuning parameter candidates Γ = {2, 3, 4, 8}

## Outputs

- Single permutation-calibrated P-value P_AMAT in [0,1] per taxa-set (e.g. written to pamat_pvalue.csv)
- Per-permutation null test statistics shared between the PSPU(γ) statistic and the final P-value
- Empirical type I error and power tables (e.g. simulation_results.csv) at the 0.05 significance level

## How to apply

Normalize the OTU count table to compositional form and OTU-wise standardize to obtain Z*, then apply Algorithm 1 screening (distance correlation learning) to keep only the selected feature set S. Compute the adaptive aSPU statistic T_AMAT on Z*_S with tuning parameter candidates Γ = {2, 3, 4, 8} inside the generalized linear model framework of the AMAT R package — linear models for continuous outcomes, logistic models for binary outcomes. Generate B = 500 permutations of the adjusted response, re-running screening and the test on each permuted dataset so the same null statistics serve both the PSPU(γ) combination step and the final P-value, reducing permutation cost. Estimate P_AMAT = (1/B) Σ I[T(b) ≤ T_AMAT]; for benchmarking against aMiSPU, OMiRKAT, OMiAT, LDM and MiHC, use 10,000 permutations for all tests and assess rejection at the nominal 0.05 level.

## Related tools

- **AMAT** (R package implementing the two-stage adaptive association test: distance-correlation feature screening followed by aSPU testing with tuning parameters Γ = {2,3,4,8} and permutation P-value calibration) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which AMAT and the competing multivariate association tests (aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) are executed) — https://github.com/kzb193/AMAT

## Examples

```
remotes::install_github("kzb193/AMAT")
```

## Evaluation signals

- The script runs without error and P_AMAT is a single numeric value in [0,1]
- Null permutation statistics are reused (shared) between the PSPU(γ) statistic and the final P-value rather than regenerated
- Under null simulation replicates (β = 0), the empirical type I error falls within the binomial 95% interval around the nominal 0.05
- All reported type I error and power values lie in [0,1]
- When benchmarking, P-values for all six tests (AMAT, aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) use the same number of permutations (10,000) on identical scenarios

## Limitations

- AMAT loses its power advantage when the association signal density is extremely sparse
- Type I error calibration depends on adequate permutations; MiHC, not AMAT, was the method that deviated (conservative, especially with binary outcomes)
- Small non-zero weights left to noise taxa by weighting-only approaches (as in existing score-statistic-weighted tests) motivated AMAT's exact-zero screening, so skipping the screening stage changes the method's behavior

## Evidence
