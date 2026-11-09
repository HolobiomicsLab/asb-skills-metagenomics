---
name: explained-variance-loss-quantification
description: Use when you need to know how much of a predictor set's (e.g., ALS vs
  Sentinel-1 radar) explanatory power for many response variables depends on between-region
  variation rather than within-region structure.
when_to_use_negative:
- When the question is absolute predictive performance of ALS vs radar (median R²,
  e.g. 0.51 vs 0.57 for first NMDS axes) rather than the region-dependent share of
  explained variance
- When the response variables come from fewer than the five forest regions, since
  region as a random intercept factor cannot be estimated
- When predictors are already region-level aggregates rather than per-plot ALS/radar
  metrics
tools:
- name: R
  role: Used to fit the fixed-effects and mixed-effects models with region as a random
    intercept factor and to compute and aggregate R² losses across the 48 response
    variables
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:cf1ad62ad2dc2c3b406b9dbd7777634d7d337086947f02f23f5b03057cb8e01c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# explained-variance-loss-quantification

## Summary

Quantifies how much explained variance (R²) is attributable to between-region differences by comparing fixed-effects biodiversity models against mixed-effects models that include the five forest regions as a random intercept, for ALS versus Sentinel-1 radar predictor sets across 48 biodiversity response variables.

## When to use

Use when you need to know how much of a predictor set's (e.g., ALS vs Sentinel-1 radar) explanatory power for many response variables depends on between-region variation rather than within-region structure.

## When NOT to use

- When the question is absolute predictive performance of ALS vs radar (median R², e.g. 0.51 vs 0.57 for first NMDS axes) rather than the region-dependent share of explained variance
- When the response variables come from fewer than the five forest regions, since region as a random intercept factor cannot be estimated
- When predictors are already region-level aggregates rather than per-plot ALS/radar metrics

## Inputs

- 48 biodiversity response variables (taxonomic and phylogenetic alpha- and beta-diversity measures across 12 functional groups)
- ALS per-plot predictor table
- Sentinel-1 radar per-plot predictor table (including plot means in S1_mean.csv)
- Region label for each of the five temperate forest regions

## Outputs

- Per-model R² for each fixed-effects and mixed-effects fit (48 models x 2 predictor sets)
- Per-model loss of explained variance (fixed-effects R² minus mixed-effects R²)
- Median loss of explained variance for ALS models and for radar models
- Full results table written to mixed_effects_model_results.csv

## How to apply

Load the 48 biodiversity response variables (taxonomic and phylogenetic alpha- and beta-diversity measures across 12 functional groups) together with the ALS and Sentinel-1 radar per-plot predictor tables, including the radar plot means in S1_mean.csv, in R. For each response variable, fit a fixed-effects model separately with ALS predictors and with radar predictors, then refit each as a mixed-effects model with region as a random intercept factor. Compute the explained variance (R²) for each fixed-effects fit and its mixed-effects counterpart, and derive the per-model loss as their difference. Aggregate the 48 per-model losses per predictor set and report the median loss separately for ALS and radar; the reported reference medians are 0.14 (ALS) and 0.10 (radar), so ALS should lose more variance than radar once region is accounted for. Write the full per-model results to mixed_effects_model_results.csv for comparison against the reference values.

## Related tools

- **R** (Used to fit the fixed-effects and mixed-effects models with region as a random intercept factor and to compute and aggregate R² losses across the 48 response variables) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
library(lme4); m_fixed <- lm(y ~ ALS_predictors, data = dat); m_mixed <- lmer(y ~ ALS_predictors + (1|region), data = dat); loss <- R2(m_fixed) - R2(m_mixed)
```

## Evaluation signals

- Median loss of explained variance for ALS models reproduces the reported 0.14 across the 48 response variables
- Median loss of explained variance for radar models reproduces the reported 0.10
- ALS median loss exceeds radar median loss, reflecting stronger region-dependence of ALS explained variance
- There are exactly 48 fixed-effects and 48 mixed-effects fits per predictor set (one per response variable)
- mixed_effects_model_results.csv contains per-model R² for both fit types and the derived per-model loss

## Limitations

- The drop in predictive power for assemblage composition of carabids, spiders and bats after accounting for regions may reflect geographical patterns or unmeasured environmental determinants rather than predictor quality
- The five study regions represent only the major temperate forest ecosystems of Central Europe, so the region-decomposition is not necessarily generalizable to other forest types

## Evidence

- [results] Accounting for region as a random factor reduced explained variance more strongly in the ALS models than in the radar models: the median loss of explained variance was 0.14 for ALS versus 0.10 for radar across the 48 response variables.: [evidence span withheld — anchor status: not_found]
- [discussion] The drop in predictive power regarding the assemblage composition of carabids, spiders and bats after accounting for regions may reflect the geographical patterns or unmeasured environmental: "The drop in predictive power regarding the assemblage composition of carabids, spiders and bats after accounting for regions may reflect the geographical patterns or unmeasured environmental"
- [methods] Like the rest of the manuscript it was performed using R.: "Like the rest of the manuscript it was performed using R."
- [intro] A suite of ground-truth taxonomic and phylogenetic biodiversity measures covering within forest stand (α-) and among forest stand (β-) diversity from a broad range of trophic levels and taxa: "A suite of ground-truth taxonomic and phylogenetic biodiversity measures covering within forest stand (α-) and among forest stand (β-) diversity from a broad range of trophic levels and taxa"
