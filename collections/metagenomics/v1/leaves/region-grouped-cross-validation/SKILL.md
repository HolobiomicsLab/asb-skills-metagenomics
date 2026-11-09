---
name: region-grouped-cross-validation
description: Use when evaluating biodiversity or ecological predictive models fitted
  to spatially clustered plots from a few study regions, to prevent spatial leakage
  between training and test folds.
when_to_use_negative:
- Plots are spatially intermingled with no regional grouping — ordinary k-fold CV
  would suffice and region-grouped folds are undefined.
- The goal is SAR data pre-processing (gamma_0 backscatter generation) rather than
  model validation — use the SNAP/GPT processing chain instead.
- Only one study region is available, leaving no between-region variation to group
  folds by.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used to fit the boosted GAMs with five-fold region-grouped cross-validation
    and compute out-of-fold R²
- name: rgeos
  role: Minimum required R package for handling plot spatial data during predictor
    assembly
- name: raster
  role: Minimum required R package for extracting radar/ALS raster statistics to 1-ha
    plots
- name: rgdal
  role: Minimum required R package for reading/writing plot shapefiles and rasters
- name: glcm
  role: Minimum required R package for texture-based predictor derivation feeding
    the models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:50c62d9362d0435016f0528cf22e9fc482c9e0d9018083f0a1f7f8b5058dad56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glcm
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# region-grouped-cross-validation

## Summary

Five-fold cross-validation in which CV folds are grouped by study region, applied to boosted GAMs (e.g. R mboost/gamboost) to obtain out-of-fold R² when comparing Sentinel-1 radar vs ALS predictors of biodiversity (NMDS composition axes, species richness, phylogenetic diversity) across 12 functional groups.

## When to use

Use when evaluating biodiversity or ecological predictive models fitted to spatially clustered plots from a few study regions, to prevent spatial leakage between training and test folds.

## When NOT to use

- Plots are spatially intermingled with no regional grouping — ordinary k-fold CV would suffice and region-grouped folds are undefined.
- The goal is SAR data pre-processing (gamma_0 backscatter generation) rather than model validation — use the SNAP/GPT processing chain instead.
- Only one study region is available, leaving no between-region variation to group folds by.

## Inputs

- Radar predictor table: plot-level means of Sentinel-1 gamma_0 time-series statistics (e.g. S1_mean.csv) for 1-ha plots
- ALS structural predictor table for the same 1-ha plots
- Ground-truth biodiversity responses for 12 functional groups: NMDS axis 1/2 scores, log-transformed species richness, phylogenetic diversity
- Plot-to-study-region assignment used to define grouped folds

## Outputs

- Per-functional-group out-of-fold R² for each response and each predictor set (radar, ALS)
- Median R² across 12 functional groups per response and predictor set
- boosted_gam_r2_summary.csv containing all per-group and median R² values

## How to apply

Fit boosted GAMs (R package mboost/gamboost) separately for the radar predictor set (plot-level means of Sentinel-1 gamma_0 time-series statistics, e.g. S1_mean.csv) and the ALS structural predictor set against each response: NMDS axis 1 and 2 scores (assemblage composition), log-transformed species richness, and phylogenetic diversity for each of 12 functional groups. Partition the 1-ha plots into five folds such that all plots from a given study region stay in the same fold, so models are tested on regions unseen during training and spatial leakage is avoided. Compute out-of-fold R² per functional group and response for both predictor sets, then take the median R² across the 12 groups for each response/predictor combination. Use identical CV configuration (same folds, same boosting settings) for radar and ALS so the comparison isolates the predictor set. Write per-group and median R² values to a summary table (boosted_gam_r2_summary.csv).

## Related tools

- **R** (Language used to fit the boosted GAMs with five-fold region-grouped cross-validation and compute out-of-fold R²)
- **rgeos** (Minimum required R package for handling plot spatial data during predictor assembly)
- **raster** (Minimum required R package for extracting radar/ALS raster statistics to 1-ha plots)
- **rgdal** (Minimum required R package for reading/writing plot shapefiles and rasters)
- **glcm** (Minimum required R package for texture-based predictor derivation feeding the models)

## Examples

```
Rscript boosted_gam_cv.R --predictors S1_mean.csv --folds region_grouped_5fold --responses nmds1,nmds2,richness,phylo_div --out boosted_gam_r2_summary.csv
```

## Evaluation signals

- Reproduced medians must match reported values: NMDS1 R² 0.51 (ALS) vs 0.57 (radar); NMDS2 0.30 vs 0.27; species richness 0.21 vs 0.11; phylogenetic diversity 0.19 vs 0.16.
- No fold contains plots from more than one study region (region-grouped folds verified).
- Radar and ALS models use identical folds and boosting configuration so R² differences reflect only the predictor set.
- boosted_gam_r2_summary.csv contains both per-group and median R² values for all 12 functional groups and all four responses.
- R² values are computed from out-of-fold predictions, not training-set fits.

## Limitations

- Radar models showed a shortcoming in predicting species richness and phylogenetic diversity for groups driven by understorey development, because C-band SAR has weak permeability through overstorey layers.
- A drop in predictive power for carabid, spider and bat assemblage composition after accounting for regions may reflect unmeasured regional determinants, so region-grouped R² can still be optimistic for such groups.
- The five study regions represent only major temperate forest ecosystems of Central Europe; generalisation to other forests is untested.

## Evidence

- [abstract] median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes"
- [discussion] In biodiversity models of 12 functional groups, radar and ALS performed equally well.: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
- [methods] Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
