---
name: external-validation-of-species-composition-models
description: Use when testing whether a fitted radar (Sentinel-1 gamma_0) assemblage-composition
  model generalizes to independent, georeferenced field plots outside the training
  regions, e.g. bird or saproxylic beetle NMDS1 scores.
when_to_use_negative:
- Validation plots lie inside the five training forest regions — that is model fitting
  / internal validation, not the independent external test this skill performs
- The response of interest is species richness or phylogenetic diversity — Sentinel-1
  performed poorly for taxa driven by understorey development (C-band weak permeability
  through overstorey layers)
- No independent, georeferenced, well-stratified species-composition data exist for
  the target area — the paper names this lack as the major barrier to larger-scale
  application
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: R
  role: 'Runtime for deriving per-plot mean backscatter statistics and for applying
    the fitted radar fixed-effects models to external plots (README S3.3: statistics
    ''performed using R'')'
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Loads the VH and VV gamma_0 temporal stacks and supports per-plot pixel extraction
    for the mean summary statistics
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgdal
  role: Reads the external plot-center shapefile (ALB_example_plots.*) and handles
    CRS alignment with the gamma_0 rasters
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgeos
  role: Geometric operations on plot polygons during per-plot extraction of radar
    predictors
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: glcm
  role: Minimum-required package for the neighborhood-based texture statistics in
    the same per-plot statistics script
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:960ca87cfedd5b9422ea06a2c09504350fcd2e9454e9f7f300b78989e4d79a71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/So-YeonBae/Sentinel1-Biodiversity
  tools:
  - R
  - glcm
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# external-validation-of-species-composition-models

## Summary

Reproduces the paper's external validation of Sentinel-1 radar assemblage-composition models: the fitted radar fixed-effects models (Supplementary Software 1) are applied, unchanged, to per-plot mean VH/VV gamma_0 backscatter statistics from independent plots outside the five training regions, and predicted vs observed NMDS1 scores are correlated (R² 0.26 for 72 bird plots, 0.22 for 91 saproxylic beetle plots).

## When to use

Use when testing whether a fitted radar (Sentinel-1 gamma_0) assemblage-composition model generalizes to independent, georeferenced field plots outside the training regions, e.g. bird or saproxylic beetle NMDS1 scores.

## When NOT to use

- Validation plots lie inside the five training forest regions — that is model fitting / internal validation, not the independent external test this skill performs
- The response of interest is species richness or phylogenetic diversity — Sentinel-1 performed poorly for taxa driven by understorey development (C-band weak permeability through overstorey layers)
- No independent, georeferenced, well-stratified species-composition data exist for the target area — the paper names this lack as the major barrier to larger-scale application

## Inputs

- Temporal stacks of VH and VV Sentinel-1 gamma_0 rasters covering the external plots (e.g., ALB_gamma0_VH.tif, ALB_gamma0_VV.tif)
- Plot-center shapefile for the external validation plots (e.g., ALB_example_plots.*)
- External species-composition matrices: 72 bird survey plots and 91 saproxylic beetle plots
- Fitted radar fixed-effects model objects from Supplementary Software 1

## Outputs

- Per-plot mean backscatter predictor table for the external plots (analogous to S1_mean.csv)
- Predicted NMDS1 assemblage-composition scores for each external plot
- Per-taxon external-validation R² (≈0.26 birds, ≈0.22 saproxylic beetles) written to external_validation_results.csv

## How to apply

Prerequisite: Sentinel-1 GRDH scenes are converted to gamma_0 via the SNAP GPT batch chain (apply precise orbit, remove border noise, remove thermal noise, radiometric calibration, radiometric terrain flattening, range-doppler terrain correction) so external predictors are computed exactly like the training data. In R (minimum packages raster, rgdal, rgeos, glcm), run the Sentinel1_pixel_ neighborhood _statistics.r script with the external plot-center shapefile and the VH/VV gamma_0 temporal stacks to compute per-plot mean backscatter summary statistics, mirroring the demo workflow that produced S1_mean.csv for 1-ha plots. Ordinate each external species-composition matrix (72 bird plots, 91 saproxylic beetle plots) with non-metric multidimensional scaling to obtain observed first-axis (NMDS1) assemblage-composition scores. Apply the fitted radar fixed-effects models from Supplementary Software 1 unchanged to the external plots' radar predictors to generate predicted NMDS1 scores — no re-fitting, so generalization beyond the five training regions is genuinely tested. Compute the coefficient of determination between predicted and observed NMDS1 separately per taxon; successful reproduction yields approximately 0.26 for birds and 0.22 for saproxylic beetles. Write the predicted NMDS1 scores and the per-taxon R² values to external_validation_results.csv.

## Related tools

- **R** (Runtime for deriving per-plot mean backscatter statistics and for applying the fitted radar fixed-effects models to external plots (README S3.3: statistics 'performed using R')) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Loads the VH and VV gamma_0 temporal stacks and supports per-plot pixel extraction for the mean summary statistics) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgdal** (Reads the external plot-center shapefile (ALB_example_plots.*) and handles CRS alignment with the gamma_0 rasters) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgeos** (Geometric operations on plot polygons during per-plot extraction of radar predictors) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **glcm** (Minimum-required package for the neighborhood-based texture statistics in the same per-plot statistics script) — https://github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
Rscript -e "source('Sentinel1_pixel_ neighborhood _statistics.r')"
```

## Evaluation signals

- Coefficient of determination between predicted and observed NMDS1 is approximately 0.26 for birds (n=72) and 0.22 for saproxylic beetles (n=91)
- All external plot centers (from the shapefile) verified to lie outside the five training forest regions
- Predictor table matches the S1_mean.csv schema: mean VH and VV gamma_0 backscatter per 1-ha plot
- Every external plot in each taxon has both an observed NMDS1 score (from NMDS ordination) and a predicted NMDS1 score, with no model re-fitting on the external data
- external_validation_results.csv contains per-plot predicted NMDS1 scores plus the two per-taxon R² values

## Limitations

- Sentinel-1 shortcoming: poor prediction of species richness and phylogenetic diversity for groups driven by understorey development, due to weak permeability of C-band SAR through overstorey layers
- Generality beyond the major temperate forest ecosystems of Central Europe (the five training regions) is untested
- Predictive power for assemblage composition dropped for carabids, spiders and bats after accounting for regions, indicating unmeasured regional determinants
- Requires independently collected, georeferenced, well-stratified species data, which the paper flags as the major barrier to larger-scale application (standardized high-resolution monitoring exists only for a few taxa)

## Evidence

- [abstract] External validation of radar models on birds and saproxylic beetles: "We further demonstrate the promising predictive ability of radar-derived data with external validation based on the species composition of birds and saproxylic beetles"
- [abstract] NMDS1 axes represent assemblage composition in radar models: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [methods] R packages required for the statistics workflow: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Summary statistics built on gamma_0 temporal stacks in R: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Per-plot (1-ha) mean outputs of the statistics script: "4) Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [readme] Statistics script to source for the per-plot workflow: "Source the R script. << Sentinel1_pixel_ neighborhood _statistics.r >>"
- [discussion] Generality limited to Central European temperate forests: "Although our study covered five forested regions, these were representative only of the major temperate forest ecosystems of Central Europe."
- [discussion] Sentinel-1 shortcoming for understorey-driven groups: "The shortcoming of Sentinel-1 data that we uncovered was in the prediction of species richness and phylogenetic diversity for groups that were more strongly driven by the development of the"
