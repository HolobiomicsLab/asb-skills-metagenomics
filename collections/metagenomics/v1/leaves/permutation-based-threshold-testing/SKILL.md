---
name: permutation-based-threshold-testing
description: Use when screening high-dimensional OTU/count features for outcome association
  by permuting an adjusted response (B = 500) to build per-feature distance-correlation
  nulls and select a data-driven testing subset S.
when_to_use_negative:
- Input is already a small prespecified taxa-set (e.g. taxa-sets restricted to at
  least five taxa) where per-feature screening adds unnecessary permutation cost
- The response is unadjusted and confounders have not yet been regressed out — the
  procedure requires an adjusted response vector
- Only a single omnibus p-value per clade is needed (e.g. FWER-controlled testing
  at 0.05 via Bonferroni across 115 genera, 57 families, 27 orders, 16 classes, 11
  phyla) without feature-level screening
edam_operation: http://edamontology.org/operation_2239
tools:
- name: AMAT
  role: Provides the distance correlation learning routine and the Algorithm 1 screening
    stage whose thresholds come from permuted response nulls
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Runtime environment in which the permutation loops, DC computation, and fallback
    selection script are executed
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:d81578b17cd9b04c8806680945c1ab2a48a9a393f64cac70ba4d34408558eb79
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# permutation-based-threshold-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds per-feature null distributions by repeatedly permuting the adjusted response and recomputing a statistic (e.g. distance correlation between each OTU abundance vector and the response), then screens features whose observed statistic exceeds the permuted mean. It is the thresholding engine behind AMAT's data-driven testing subset S selection.

## When to use

Use when screening high-dimensional OTU/count features for outcome association by permuting an adjusted response (B = 500) to build per-feature distance-correlation nulls and select a data-driven testing subset S.

## When NOT to use

- Input is already a small prespecified taxa-set (e.g. taxa-sets restricted to at least five taxa) where per-feature screening adds unnecessary permutation cost
- The response is unadjusted and confounders have not yet been regressed out — the procedure requires an adjusted response vector
- Only a single omnibus p-value per clade is needed (e.g. FWER-controlled testing at 0.05 via Bonferroni across 115 genera, 57 families, 27 orders, 16 classes, 11 phyla) without feature-level screening

## Inputs

- n x p OTU count matrix (e.g. 856 OTUs x 60 throat microbiome samples, or multinomial/Dirichlet-multinomial simulations)
- adjusted response vector (continuous from a linear model or binary from a logistic model)
- number of permutations B (e.g. B = 500 for null DC distributions)

## Outputs

- Selected feature index set S, a non-empty subset of {1,...,p} written to a results file
- Per-feature permuted distance-correlation null distributions (B values per feature)
- Screening script and downstream empirical type I error / power table (e.g. simulation_results.csv at nominal 0.05)

## How to apply

For each feature j = 1..p (e.g. 856 OTUs mimicking the throat microbiome), compute the observed distance correlation dcj between the feature's abundance vector and the adjusted response under the generalized linear model framework. Permute the response B times (B = 500 in the simulation workflow; 10 000 permutations were used for reported p-values of AMAT and competitors) and recompute every feature's distance correlation on each permuted dataset to form a per-feature null distribution. Select features whose observed dcj exceeds the mean of its B permuted values; apply AMAT's fallback rule when nothing passes, selecting the single feature with maximum observed distance correlation so S is always non-empty. Tuning candidates for the subsequent data-adaptive test were Gamma = {2, 3, 4, 8} with the kzb193/AMAT R implementation.

## Related tools

- **AMAT** (Provides the distance correlation learning routine and the Algorithm 1 screening stage whose thresholds come from permuted response nulls) — https://github.com/kzb193/AMAT
- **R** (Runtime environment in which the permutation loops, DC computation, and fallback selection script are executed) — https://github.com/kzb193/AMAT

## Examples

```
Rscript -e 'devtools::install_github("kzb193/AMAT"); library(AMAT)'
```

## Evaluation signals

- Selected set S is a valid non-empty subset of feature indices {1..p} and is written to the results file
- The fallback selection fires whenever the mean-threshold rule selects nothing, guaranteeing S is never empty
- Downstream empirical type I error falls within the binomial 95% interval around the nominal 0.05 across all simulated configurations (continuous and binary outcomes)
- All reported power and type I error values lie in [0,1]
- Permutation nulls are actually built with the specified B (e.g. 500) per feature, not a single global null

## Limitations

- If no feature's observed distance correlation exceeds its permuted mean, the fallback picks a single max-DC feature that may carry no true signal
- Signal sparsity matters: with only 10-30% of features associated, mean-threshold screening trades off false exclusions; naive aggregation of low-rank taxa can lose power when opposite-direction effects cancel
- Reported type I error control was validated under multinomial OTU generation mimicking the throat microbiome (856 OTUs); other count-generating regimes may not preserve nominal levels

## Evidence
