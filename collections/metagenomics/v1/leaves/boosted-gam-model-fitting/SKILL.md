---
name: boosted-gam-model-fitting
description: Use when tabular plot-level predictor sets (e.g., Sentinel-1 gamma_0
  time-series statistics vs ALS structural metrics) must be compared for modelling
  biodiversity responses such as NMDS axes, species richness, or phylogenetic diversity
  with cross-validated regression.
when_to_use_negative:
- The inputs are still raw Sentinel-1 GRDH scenes — this skill consumes plot-level
  predictor tables produced after the SNAP/GPT gamma_0 processing chain (Apply precise
  orbit → Remove border noise → Remove thermal noise → Radiometric calibration → Radiometric
  terrain flattening → Range-doppler terrain correction) and the R pixel/neighbourhood
  summary statistics.
- The goal is external validation on independent data collected outside the five training
  regions (e.g., bird and saproxylic beetle species composition) rather than cross-validated
  comparison on the training plots.
- The target is area-wide biodiversity mapping or monitoring of biodiversity change
  rather than quantifying per-plot predictive power of a predictor set for the 12
  functional groups.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: R
  role: Language/environment used to fit the boosted GAMs, run the five-fold region-grouped
    cross-validation, and compute out-of-fold and median R²
  repo: https://cran.r-project.org/
- name: mboost (gamboost)
  role: Implements the boosted generalized additive models fitted against each biodiversity
    response for the radar and ALS predictor sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:2a7416b4d51e383e3fa722b2258761e64fef33ad9cc72dfd689d6e1311c66053
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mboost (gamboost)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# boosted-gam-model-fitting

## Summary

Cross-validated boosted generalized additive models (R, mboost/gamboost) that model NMDS1/2 assemblage-composition scores, log-transformed species richness, and phylogenetic diversity for 12 functional groups from either Sentinel-1 gamma_0 time-series predictors or ALS structural predictors. Its five-fold region-grouped CV and out-of-fold R² medians (NMDS1: 0.57 radar vs 0.51 ALS) are the quantitative basis for the paper's claim that radar performs as well as ALS.

## When to use

Use when tabular plot-level predictor sets (e.g., Sentinel-1 gamma_0 time-series statistics vs ALS structural metrics) must be compared for modelling biodiversity responses such as NMDS axes, species richness, or phylogenetic diversity with cross-validated regression.

## When NOT to use

- The inputs are still raw Sentinel-1 GRDH scenes — this skill consumes plot-level predictor tables produced after the SNAP/GPT gamma_0 processing chain (Apply precise orbit → Remove border noise → Remove thermal noise → Radiometric calibration → Radiometric terrain flattening → Range-doppler terrain correction) and the R pixel/neighbourhood summary statistics.
- The goal is external validation on independent data collected outside the five training regions (e.g., bird and saproxylic beetle species composition) rather than cross-validated comparison on the training plots.
- The target is area-wide biodiversity mapping or monitoring of biodiversity change rather than quantifying per-plot predictive power of a predictor set for the 12 functional groups.

## Inputs

- Plot-level means of Sentinel-1 gamma_0 time-series statistics (radar predictor table, e.g., S1_mean.csv) for the 1-ha plots
- ALS forest-structure predictor table for the same 1-ha plots
- Ground-truth biodiversity responses per functional group: NMDS axis 1 and 2 scores (assemblage composition), log-transformed species richness, phylogenetic diversity
- Study-region label for each plot (five temperate forest regions) used to build region-grouped CV folds

## Outputs

- Out-of-fold R² per functional group (12 groups) for each response (NMDS1, NMDS2, richness, phylogenetic diversity) under each predictor set (radar, ALS)
- Median R² across the 12 functional groups for each response and predictor set
- boosted_gam_r2_summary.csv containing all per-group and median R² values

## How to apply

Assemble one plot-level design table per predictor set: radar predictors are plot-level means of Sentinel-1 gamma_0 time-series statistics (e.g., S1_mean.csv) and ALS predictors are the structural metrics for the same 1-ha plots, each joined to the ground-truth responses (NMDS axis 1 and 2 scores, log-transformed species richness, phylogenetic diversity) for each of the 12 functional groups. Fit boosted GAMs with the R package mboost/gamboost — componentwise boosting regularizes the spline-based GAM and keeps out-of-fold performance honest at modest plot counts. Run five-fold cross-validation in which folds are grouped by study region (the five temperate forest regions) so no plot from a training region leaks into the validation fold, and reuse the identical CV configuration for the ALS predictor set so the radar-vs-ALS comparison is paired. Compute out-of-fold R² per functional group and response for both predictor sets, then take the median R² across the 12 groups for each response and predictor set. Write all per-group and median R² values to boosted_gam_r2_summary.csv and check them against the reported medians: NMDS1 0.51 (ALS) vs 0.57 (radar), NMDS2 0.30 vs 0.27, species richness 0.21 vs 0.11, phylogenetic diversity 0.19 vs 0.16.

## Related tools

- **R** (Language/environment used to fit the boosted GAMs, run the five-fold region-grouped cross-validation, and compute out-of-fold and median R²) — https://cran.r-project.org/
- **mboost (gamboost)** (Implements the boosted generalized additive models fitted against each biodiversity response for the radar and ALS predictor sets)

## Examples

```
library(mboost); fit <- gamboost(NMDS1 ~ ., data = model_frame)
```

## Evaluation signals

- boosted_gam_r2_summary.csv contains per-group R² values for all 12 functional groups × 4 responses × 2 predictor sets plus the median row.
- Computed medians reproduce the reported values: NMDS1 0.51 (ALS) vs 0.57 (radar), NMDS2 0.30 vs 0.27, species richness 0.21 vs 0.11, phylogenetic diversity 0.19 vs 0.16.
- Fold assignment is grouped by study region — no plot from a region appears in both training and validation folds (no spatial leakage).
- R² values are computed from out-of-fold (held-out) predictions, not on training data; training-set R² should be noticeably higher than out-of-fold R².
- Radar and ALS median R² are comparable for the NMDS axes (0.57 vs 0.51), consistent with 'radar and ALS performed equally well'.

## Limitations

- Sentinel-1 C-band data fall short for species richness and phylogenetic diversity of groups driven by understorey development (richness median R² only 0.11 radar vs 0.21 ALS) because of weak permeability of short-wavelength C-band SAR through overstorey layers.
- Phylogenetic diversity of higher trophic groups was not well predicted, as its determinants were associated with understorey density.
- Predictive power drops for carabid, spider and bat assemblage composition after accounting for regions, suggesting unmeasured regional determinants.
- The five study regions represent only the major temperate forest ecosystems of Central Europe; generality in other forest types is untested.

## Evidence

- [abstract] Headline result: median R² over twelve taxa, ALS 0.51 vs radar 0.57 for first NMDS axes: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [intro] Scope of the modelling exercise: composition, richness, phylogenetic diversity, radar vs ALS: "Thus, we quantify the predictive power of radar in modelling different aspects of biodiversity, including species composition and richness and phylogenetic diversity"
- [discussion] Equal performance of radar and ALS across the 12 functional groups: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
- [discussion] Known radar shortcoming for understorey-driven richness and phylogenetic diversity: "The shortcoming of Sentinel-1 data that we uncovered was in the prediction of species richness and phylogenetic diversity for groups that were more strongly driven by the development of the"
- [readme] Analyses in this manuscript were performed in R: "Like the rest of the manuscript it was performed using R."
