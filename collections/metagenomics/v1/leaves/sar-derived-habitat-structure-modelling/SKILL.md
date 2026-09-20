---
name: sar-derived-habitat-structure-modelling
description: Use when modelling ground-truth alpha/beta diversity of forest taxa from
  per-plot Sentinel-1 gamma0 radar or ALS habitat-structure predictors, especially
  when comparing fixed-effects vs mixed-effects fits with the five forest regions
  as a random factor.
when_to_use_negative:
- Input data are raw Sentinel-1 GRDH slant-range products or unprocessed temporal
  gamma0 stacks rather than per-plot predictor tables — use the SAR pre-processing/summary-statistics
  skill first.
- The goal is external validation of radar models on data collected outside the five
  training regions rather than within-region modelling with a region random factor.
- Response variables or plots lack region labels, so region cannot be included as
  a random intercept factor.
edam_operation: http://edamontology.org/operation_2239
tools:
- name: R
  role: Environment for fitting fixed-effects and mixed-effects models (region as
    random intercept) and computing explained variance for all 48 response variables
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: R package for handling the temporal gamma0 raster stacks behind the per-plot
    radar predictors used in the models
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: glcm
  role: R package used for texture-based neighborhood statistics contributing to the
    radar habitat-structure predictors
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:3f588c9e5da9403dd6157bd0004e5e84de07dc66529667b107fd3851c3490e73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glcm
  - raster
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sar-derived-habitat-structure-modelling

## Summary

Builds biodiversity models for 48 taxonomic/phylogenetic alpha- and beta-diversity response variables across 12 functional groups using per-plot Sentinel-1 gamma0 radar or ALS structural predictors, and quantifies how much explained variance (R-squared) is lost when region is added as a random intercept in mixed-effects models.

## When to use

Use when modelling ground-truth alpha/beta diversity of forest taxa from per-plot Sentinel-1 gamma0 radar or ALS habitat-structure predictors, especially when comparing fixed-effects vs mixed-effects fits with the five forest regions as a random factor.

## When NOT to use

- Input data are raw Sentinel-1 GRDH slant-range products or unprocessed temporal gamma0 stacks rather than per-plot predictor tables — use the SAR pre-processing/summary-statistics skill first.
- The goal is external validation of radar models on data collected outside the five training regions rather than within-region modelling with a region random factor.
- Response variables or plots lack region labels, so region cannot be included as a random intercept factor.

## Inputs

- 48 taxonomic and phylogenetic alpha- and beta-diversity response variables across 12 functional groups (including first NMDS axes of assemblage composition)
- Per-plot ALS predictor table (very high density airborne laser scanning metrics)
- Per-plot Sentinel-1 radar predictor table of temporal gamma0 summary statistics (radar plot means provided in S1_mean.csv)
- Region labels assigning each 1-ha plot to one of the five temperate forest regions

## Outputs

- R-squared per fixed-effects model and per mixed-effects model for each of the 48 response variables
- Per-model loss of explained variance (fixed-effects R-squared minus mixed-effects R-squared) for ALS and radar model sets
- Median loss of explained variance reported separately for ALS models and radar models (expected ~0.14 ALS vs ~0.10 radar)
- Full results table written to mixed_effects_model_results.csv

## How to apply

Load the 48 biodiversity response variables together with the ALS and Sentinel-1 radar per-plot predictor tables (radar plot means supplied as S1_mean.csv) into R. For each response variable, fit a separate fixed-effects model with the ALS predictors and with the radar predictors, then refit each as a mixed-effects model with region as a random intercept factor. Compute explained variance (R-squared) for every fixed-effects and mixed-effects fit and take their difference as the per-model loss of explained variance. Aggregate losses across all 48 response variables and report the median loss separately for ALS models and radar models; reference medians are 0.14 (ALS) and 0.10 (radar). Write the full results table to mixed_effects_model_results.csv and compare reproduced medians against the reported reference values.

## Related tools

- **R** (Environment for fitting fixed-effects and mixed-effects models (region as random intercept) and computing explained variance for all 48 response variables) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (R package for handling the temporal gamma0 raster stacks behind the per-plot radar predictors used in the models) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **glcm** (R package used for texture-based neighborhood statistics contributing to the radar habitat-structure predictors) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
m_fixed <- lm(alpha ~ vh_median + vv_median, data = plots); m_mixed <- lmer(alpha ~ vh_median + vv_median + (1 | region), data = plots)
```

## Evaluation signals

- Median loss of explained variance across the 48 response variables reproduces the reference values: 0.14 for ALS models and 0.10 for radar models.
- Per-model loss of explained variance equals fixed-effects R-squared minus mixed-effects R-squared and is non-negative for each of the 48 response variables.
- Baseline fixed-effects performance is in the expected range: median R-squared over twelve taxa for the first NMDS axes is ~0.51 (ALS) and ~0.57 (radar).
- Every one of the 48 response variables has both a fixed-effects and a mixed-effects fit in the results table.
- The full results table exists at mixed_effects_model_results.csv with per-model R-squared and loss values for both ALS and radar model sets.

## Limitations

- The drop in predictive power for the assemblage composition of carabids, spiders and bats after accounting for regions may reflect geographical patterns or unmeasured environmental determinants.
- Sentinel-1 C-band radar has weak permeability through overstorey layers, so it poorly predicts species richness and phylogenetic diversity for groups driven by understorey development, and phylogenetic diversity of higher trophic groups was not well predicted.
- The five study regions represent only the major temperate forest ecosystems of Central Europe; generality in other forest types is untested.

## Evidence

- [results] median loss of explained variance ALS vs radar: [evidence span withheld — anchor status: not_found]
- [abstract] median R-squared for NMDS axes: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes"
- [methods] R used throughout: "Like the rest of the manuscript it was performed using R."
- [readme] radar plot means file: "4) Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [readme] required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
