---
name: radar-fixed-effects-model-prediction
description: Use when you have a fitted radar biodiversity model and must predict
  assemblage composition (NMDS1) on plots outside the five training regions, then
  externally validate against observed ordination scores.
when_to_use_negative:
- When modelling species richness or phylogenetic diversity for taxa driven by understorey
  development, which Sentinel-1 C-band SAR predicts poorly due to weak permeability
  through overstorey layers
- When the target plots fall within the five training regions — use in-sample model
  fitting/evaluation instead of external validation
- When predictions require regional random effects; this skill applies the fixed-effects
  component only for extrapolation beyond training regions
tools:
- name: R
  role: Runtime for extracting plot-level radar statistics, running the NMDS ordination
    of external composition matrices, applying the fixed-effects models and computing
    R²
- name: raster
  role: Read the VH/VV gamma_0 temporal stacks and summarize per-plot mean backscatter
    for external plots
- name: rgdal
  role: Read the external plot-center shapefile and handle coordinate reference systems
- name: rgeos
  role: Geometric operations linking plot-center geometries to radar raster pixels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:f921cb081dea41e22de6932f6aa7212bc7f97292bc247ccc742796a429fc1094
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - raster
  - rgdal
  - rgeos
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# radar-fixed-effects-model-prediction

## Summary

Applies the fitted Sentinel-1 radar fixed-effects models (Supplementary Software 1) to radar predictors of independent external plots to predict NMDS1 assemblage-composition scores, then scores them against observed NMDS1 via the coefficient of determination (R² ≈ 0.26 birds, 0.22 saproxylic beetles).

## When to use

Use when you have a fitted radar biodiversity model and must predict assemblage composition (NMDS1) on plots outside the five training regions, then externally validate against observed ordination scores.

## When NOT to use

- When modelling species richness or phylogenetic diversity for taxa driven by understorey development, which Sentinel-1 C-band SAR predicts poorly due to weak permeability through overstorey layers
- When the target plots fall within the five training regions — use in-sample model fitting/evaluation instead of external validation
- When predictions require regional random effects; this skill applies the fixed-effects component only for extrapolation beyond training regions

## Inputs

- Temporal stacks of VH and VV Sentinel-1 gamma_0 GeoTIFF data
- Plot-center shapefile for external validation plots
- External species composition matrices (72 bird plots, 91 saproxylic beetle plots)
- Fitted radar fixed-effects models from Supplementary Software 1

## Outputs

- Predicted NMDS1 assemblage-composition scores per external plot and taxon
- External validation coefficients of determination (R² ≈ 0.26 birds, 0.22 saproxylic beetles)
- external_validation_results.csv

## How to apply

Compute per-plot mean backscatter summary statistics for the external validation plots from temporal stacks of VH and VV Sentinel-1 gamma_0 data and the plot-center shapefile (raster, rgdal, rgeos). Ordinate each external species composition matrix (72 bird plots, 91 saproxylic beetle plots) by non-metric multidimensional scaling to obtain observed NMDS1 scores. Apply the fitted radar fixed-effects models from Supplementary Software 1 to the external plots' radar predictors (fixed effects only, no regional random effects) to generate predicted NMDS1 scores. Compute the coefficient of determination between predicted and observed NMDS1 separately per taxon, expecting approximately 0.26 for birds and 0.22 for saproxylic beetles. Write predicted scores and R² values to external_validation_results.csv.

## Related tools

- **R** (Runtime for extracting plot-level radar statistics, running the NMDS ordination of external composition matrices, applying the fixed-effects models and computing R²)
- **raster** (Read the VH/VV gamma_0 temporal stacks and summarize per-plot mean backscatter for external plots)
- **rgdal** (Read the external plot-center shapefile and handle coordinate reference systems)
- **rgeos** (Geometric operations linking plot-center geometries to radar raster pixels)

## Examples

```
predicted_nmds1 <- predict(radar_fixed_effects_model, newdata = external_radar_predictors); external_validation_results.csv
```

## Evaluation signals

- Coefficient of determination between predicted and observed NMDS1 is approximately 0.26 for birds (n=72) and 0.22 for saproxylic beetles (n=91)
- Predictions are generated on plots outside the five training forest regions, confirming genuine external validation
- Predicted NMDS1 scores fall on the same ordination scale as observed scores (no rescaling needed for R² computation)
- external_validation_results.csv contains both predicted scores and per-taxon R² values
- Per-plot radar summary statistics match the mean-backscatter scheme used to fit the training models (per-plot means over 1-ha plots)

## Limitations

- C-band space-borne SAR has weak permeability through overstorey layers, so signals tied to understorey development transfer poorly
- The five training regions represent only major temperate forest ecosystems of Central Europe; generality in other forest types is untested
- External validation was demonstrated only for two taxa (birds and saproxylic beetles); other taxa lack external validation

## Evidence

- [abstract] External validation finding: "We further demonstrate the promising predictive ability of radar-derived data with external validation based on the species composition of birds and saproxylic beetles"
- [intro] External validation scope: "the radar models for two taxa are validated using independent external data collected from areas outside the five training"
- [readme] R-based statistics workflow: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
