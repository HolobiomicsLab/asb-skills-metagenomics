---
name: mixed-effects-model-fitting-region-random-effects
description: Use when repeated measures within a small number of study regions (e.g.,
  five temperate forest regions) may inflate fixed-effects R², and you need to quantify
  the per-model loss of explained variance from adding region as a random intercept.
when_to_use_negative:
- The predictor table is already a processed feature table with no per-plot response
  data — the skill requires paired response/predictor plots with a region label
- The study design has many regions or independent sampling units, so random-intercept
  adjustment for region is not the relevant source of shared variance
- The goal is Sentinel-1 SAR preprocessing (orbit correction, thermal noise removal,
  terrain flattening) — use the SNAP/GPT preprocessing skill instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for fitting the fixed-effects and mixed-effects models and computing
    explained variance (R²) for the 48 response variables
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: S1_mean.csv
  role: Supplied per-plot mean Sentinel-1 radar predictor values for all 1-ha plots,
    used as the radar predictor table
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:30489e017b84c4ed7df3595b60ab95da44d3798f9c2fe2de5e1c0180bfda1623
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - S1_mean.csv
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mixed-effects-model-fitting-region-random-effects

## Summary

Fits per-response-variable biodiversity models (taxonomic and phylogenetic α- and β-diversity across 12 functional groups) in R with ALS or Sentinel-1 radar predictors, then refits them as mixed-effects models with forest region as a random intercept to quantify how much explained variance (R²) is lost after accounting for region.

## When to use

Use when repeated measures within a small number of study regions (e.g., five temperate forest regions) may inflate fixed-effects R², and you need to quantify the per-model loss of explained variance from adding region as a random intercept.

## When NOT to use

- The predictor table is already a processed feature table with no per-plot response data — the skill requires paired response/predictor plots with a region label
- The study design has many regions or independent sampling units, so random-intercept adjustment for region is not the relevant source of shared variance
- The goal is Sentinel-1 SAR preprocessing (orbit correction, thermal noise removal, terrain flattening) — use the SNAP/GPT preprocessing skill instead

## Inputs

- 48 biodiversity response variables (taxonomic and phylogenetic alpha- and beta-diversity measures across 12 functional groups)
- Per-plot ALS predictor table
- Per-plot Sentinel-1 radar predictor table (plot means provided in S1_mean.csv)
- Region assignment factor for each 1-ha plot (five forest regions)

## Outputs

- Per-model R² for each fixed-effects and mixed-effects fit
- Per-model loss of explained variance (fixed-effects R² minus mixed-effects R²)
- Median loss of explained variance for ALS models and for radar models
- Full results table written to mixed_effects_model_results.csv

## How to apply

Load the 48 biodiversity response variables together with the per-plot ALS and Sentinel-1 radar predictor tables (radar plot means supplied as S1_mean.csv) into R. For each response variable, fit a fixed-effects model with ALS predictors and one with radar predictors, then refit each as a mixed-effects model adding region as a random intercept factor. Compute explained variance (R²) for each fixed-effects and corresponding mixed-effects fit and derive the per-model loss as their difference. Aggregate the 48 per-model losses and report the median separately for ALS and radar models; in this study the median loss was 0.14 for ALS versus 0.10 for radar, with notable drops for carabid, spider and bat assemblage composition attributed to unmeasured regional determinants.

## Related tools

- **R** (Environment for fitting the fixed-effects and mixed-effects models and computing explained variance (R²) for the 48 response variables) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **S1_mean.csv** (Supplied per-plot mean Sentinel-1 radar predictor values for all 1-ha plots, used as the radar predictor table) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
Rscript -e 'library(lme4); m_fixed <- lm(response ~ ., data=plot_df); m_mixed <- lmer(response ~ . + (1|region), data=plot_df)'
```

## Evaluation signals

- Reproduced median loss of explained variance matches reported reference values: 0.14 for ALS models and 0.10 for radar models across the 48 response variables
- Each of the 48 response variables yields exactly one fixed-effects R², one mixed-effects R², and one derived per-model loss (48 rows per model type in mixed_effects_model_results.csv)
- Radar model losses should be systematically smaller than ALS model losses (median 0.10 vs 0.14)
- Largest R² drops are expected for carabid, spider and bat assemblage composition, consistent with the paper's regional-determinants discussion
- Each mixed-effects fit includes region only as a random intercept, not as a fixed predictor

## Limitations

- The drop in predictive power for carabid, spider and bat assemblage composition after accounting for regions may reflect geographical patterns or unmeasured environmental determinants
- The five forest regions are representative only of major temperate forest ecosystems of Central Europe, so region-level variance components may not generalize to other forests
- Loss of explained variance after adding a random intercept reflects shared regional structure, not a change in underlying predictor quality

## Evidence

- [discussion] The drop in predictive power regarding the assemblage composition of carabids, spiders and bats after accounting for regions may reflect the geographical patterns or unmeasured environmental: "The drop in predictive power regarding the assemblage composition of carabids, spiders and bats after accounting for regions may reflect the geographical patterns or unmeasured environmental"
- [methods] Like the rest of the manuscript it was performed using R.: "Like the rest of the manuscript it was performed using R."
- [readme] Calculated mean values of all our 1-ha plots: S1_mean.csv: "Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [discussion] In biodiversity models of 12 functional groups, radar and ALS performed equally well.: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
