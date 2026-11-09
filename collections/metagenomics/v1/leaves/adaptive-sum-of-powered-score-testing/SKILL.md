---
name: adaptive-sum-of-powered-score-testing
description: 'Use when testing H0: β = 0 for multivariate association between a selected
  subset of microbial taxa and an outcome in high-dimensional compositional OTU data,
  where the number of taxa (e.g. 856 OTUs) exceeds the sample size.'
when_to_use_negative:
- When the goal is single-taxon univariate association rather than a multivariate
  clade/community test
- When no feature screening is desired and all p taxa should enter the multivariate
  statistic with non-zero weights (AMAT's premise is shrinking noise features to exactly
  zero)
- When the taxa-set being tested contains fewer than five taxa, since the paper restricted
  association analysis to taxa-sets with at least five taxa
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: AMAT
  role: R package implementing the two-stage feature-selection-plus-aSPU association
    test; computes T_AMAT and permutation-based P-values
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which the AMAT implementation is executed for the
    adaptive sum of powered score test
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:cc9f39fe4b1d64aa4e0970c649268a38ff376ef101089f800caf5521c991c4fd
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
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# adaptive-sum-of-powered-score-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

AMAT's two-stage testing procedure: after distance-correlation feature selection shrinks noise taxa to zero weight, an adaptive sum of powered score (aSPU) statistic with a power set Γ = {2,3,4,8} is computed on the selected subset under a generalized linear model, with permutation-based calibration shared between the PSPU(γ) combination step and the final P-value.

## When to use

Use when testing H0: β = 0 for multivariate association between a selected subset of microbial taxa and an outcome in high-dimensional compositional OTU data, where the number of taxa (e.g. 856 OTUs) exceeds the sample size.

## When NOT to use

- When the goal is single-taxon univariate association rather than a multivariate clade/community test
- When no feature screening is desired and all p taxa should enter the multivariate statistic with non-zero weights (AMAT's premise is shrinking noise features to exactly zero)
- When the taxa-set being tested contains fewer than five taxa, since the paper restricted association analysis to taxa-sets with at least five taxa

## Inputs

- OTU abundance matrix (e.g. 856 OTUs x 60 samples for the throat microbiome data set, 557 OTUs x 98 samples for the gut data set)
- Adjusted response vector (outcome variable for the GLM)
- Selected feature set S obtained from the upstream Algorithm 1 screening sub-task on Z*

## Outputs

- P_AMAT: a single numeric P-value in [0,1] written to pamat_pvalue.csv
- Permutation null test statistics T(b) reused for both the PSPU(γ) statistic and the final P-value

## How to apply

After the upstream Algorithm 1 screening step has produced the selected feature set S from the standardized compositional matrix Z*, compute the aSPU statistic T_AMAT on Z*_S using powers Γ = {2,3,4,8} within the generalized linear model framework of the AMAT R package. Generate B = 500 permutations of the adjusted response vector; for each permutation, re-run the screening step and the adaptive test so the same set of null statistics serves both the PSPU(γ) combination and the final P-value, which greatly reduces computational cost. Estimate P_AMAT = (1/B) Σ I[T(b) ≤ T_AMAT] over the 500 null test statistics. Rationale: feature selection shrinks noise taxa to exactly zero weight, avoiding the noise accumulation that degrades score-weighted tests, while permutation calibration preserves correct type I error.

## Related tools

- **AMAT** (R package implementing the two-stage feature-selection-plus-aSPU association test; computes T_AMAT and permutation-based P-values) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which the AMAT implementation is executed for the adaptive sum of powered score test) — https://github.com/kzb193/AMAT

## Examples

```
Rscript amat_test.R --otu otu_table.csv --response adjusted_response.csv --selected S.rds --gamma 2,3,4,8 --nperm 500 --out pamat_pvalue.csv
```

## Evaluation signals

- The script runs without error end-to-end on the OTU matrix and adjusted response vector
- P_AMAT is a single numeric value in the range [0,1]
- The null permutation statistics are demonstrably shared between the PSPU(γ) statistic and the final P-value (reduced permutation cost)
- B = 500 permutations of the adjusted response vector are generated and each permutation re-runs both screening and testing
- P_AMAT is computed as (1/B) Σ I[T(b) ≤ T_AMAT] over the 500 null statistics and written to pamat_pvalue.csv

## Limitations

- Only 500 permutations are used for calibration, so the minimum attainable P-value is bounded by 1/500 (fine at α = 0.05 but limiting for genome-wide-style correction thresholds)
- Power depends on quality of the upstream feature selection; taxa not selected into S contribute zero weight by construction
- Type I error control was well maintained in the paper's simulations but MiHC was conservative, particularly with binary outcomes, in comparisons with AMAT

## Evidence

- [abstract] AMAT first alleviates the burden of noise accumulation via distance correlation learning, and then conducts a data-adaptive association test under the flexible generalized linear model framework.: "AMAT first alleviates the burden of noise accumulation via distance correlation learning, and then conducts a data-adaptive association test under the flexible generalized linear model framework."
- [abstract] Extensive simulation studies and real data applications demonstrate that AMAT is highly robust and often more powerful than several existing methods, while preserving the correct type I error rate.: "Extensive simulation studies and real data applications demonstrate that AMAT is highly robust and often more powerful than several existing methods, while preserving the correct type I error rate."
- [results] AMAT had powers of 44.7%, 49.8% and 52.5% under signal densities 10%, 20% and 30%, respectively, while the corresponding powers of the first runner-ups were only 33.6%: "AMAT had powers of 44.7%, 49.8% and 52.5% under signal densities 10%, 20% and 30%, respectively, while the corresponding powers of the first runner-ups were only 33.6%"
- [other] A free implementation of AMAT in R computing environment is available at https://github.com/kzb193/AMAT.: "A free implementation of AMAT in R computing environment is available at https://github.com/kzb193/AMAT."
