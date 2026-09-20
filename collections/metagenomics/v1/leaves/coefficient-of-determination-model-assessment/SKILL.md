---
name: coefficient-of-determination-model-assessment
description: Use when a fitted model (e.g. radar fixed-effects NMDS1 models) must
  be validated on independent external plots, and predicted-vs-observed agreement
  needs to be quantified as R² per taxon.
when_to_use_negative:
- Evaluating radar prediction of species richness or phylogenetic diversity for taxa
  driven by understorey development — C-band Sentinel-1 has weak permeability through
  overstorey layers and was shown to underperform for these biodiversity measures
- Extending the assessment to forest ecosystems outside temperate Central Europe —
  the five study regions only represent major temperate forest ecosystems of Central
  Europe, so generality elsewhere is untested
- Input is already a within-training-region model fit (use internal cross-validation
  there); this skill targets independent external validation data collected outside
  the training regions
tools:
- name: R
  role: Ordinate external species matrices via NMDS, apply the fitted radar fixed-effects
    models to external plot predictors, compute predicted-vs-observed R² per taxon,
    and write external_validation_results.csv
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster / rgdal / rgeos / glcm
  role: R packages used to load VH/VV gamma_0 temporal stacks and plot-center shapefiles
    and compute per-plot mean backscatter summary statistics feeding the validation
    predictors
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:24f5ce004b4170b13ec8e519ac705e714f55d60735f130fc6f1e2b8dbabb856c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - raster / rgdal / rgeos / glcm
  - rgdal
  - rgeos
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# coefficient-of-determination-model-assessment

## Summary

Assesses predictive performance of biodiversity models by computing the coefficient of determination (R²) between model-predicted and independently observed values — here, radar-model-predicted NMDS1 assemblage-composition scores against externally surveyed bird and saproxylic beetle plots outside the five training regions.

## When to use

Use when a fitted model (e.g. radar fixed-effects NMDS1 models) must be validated on independent external plots, and predicted-vs-observed agreement needs to be quantified as R² per taxon.

## When NOT to use

- Evaluating radar prediction of species richness or phylogenetic diversity for taxa driven by understorey development — C-band Sentinel-1 has weak permeability through overstorey layers and was shown to underperform for these biodiversity measures
- Extending the assessment to forest ecosystems outside temperate Central Europe — the five study regions only represent major temperate forest ecosystems of Central Europe, so generality elsewhere is untested
- Input is already a within-training-region model fit (use internal cross-validation there); this skill targets independent external validation data collected outside the training regions

## Inputs

- External species composition matrices for 72 bird plots and 91 saproxylic beetle plots (plots outside the five training forest regions)
- Fitted radar fixed-effects models for NMDS1 assemblage composition (Supplementary Software 1)
- Per-plot mean backscatter summary statistics computed from temporal stacks of VH and VV Sentinel-1 gamma_0 data (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) at external plot centers
- Shapefile of external plot centers (e.g. ALB_example_plots.*)

## Outputs

- Predicted NMDS1 assemblage-composition scores for each external plot and taxon
- Validation R² values per taxon (expected ≈0.26 birds, ≈0.22 saproxylic beetles)
- external_validation_results.csv

## How to apply

First, ordinate the external species composition matrices (72 bird plots, 91 saproxylic beetle plots) by non-metric multidimensional scaling to obtain observed NMDS1 assemblage-composition scores. Next, apply the fitted radar fixed-effects models (Supplementary Software 1) to the external plots' radar predictors — per-plot mean backscatter summary statistics derived from temporal stacks of VH and VV Sentinel-1 gamma_0 data at the plot-center shapefile locations — to generate predicted NMDS1 scores. Then compute the coefficient of determination between predicted and observed NMDS1 scores separately for each taxon, treating R² ≈ 0.26 for birds and R² ≈ 0.22 for saproxylic beetles as the expected benchmark values from the paper's external validation. Report R² alongside sample sizes (n=72 birds, n=91 beetles) and write predicted scores and R² values to external_validation_results.csv. Use these R² values to judge whether the radar assemblage-composition models generalize beyond the training regions.

## Related tools

- **R** (Ordinate external species matrices via NMDS, apply the fitted radar fixed-effects models to external plot predictors, compute predicted-vs-observed R² per taxon, and write external_validation_results.csv) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster / rgdal / rgeos / glcm** (R packages used to load VH/VV gamma_0 temporal stacks and plot-center shapefiles and compute per-plot mean backscatter summary statistics feeding the validation predictors) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
r2 <- cor(predicted_nmds1, observed_nmds1)^2  # per taxon: expect ~0.26 birds (n=72), ~0.22 beetles (n=91); write to external_validation_results.csv
```

## Evaluation signals

- Bird validation R² ≈ 0.26 and saproxylic beetle R² ≈ 0.22 on plots outside the five training regions
- R² is bounded in [0, 1]; a negative or near-zero value signals misapplied model or mismatched predictors
- Predicted NMDS1 scores must exist for every external plot (n=72 birds, n=91 beetles) — no missing predictions
- As a cross-check on model class, radar's median R² over twelve taxa for first NMDS axes is 0.57 vs 0.51 for ALS, so external-validation R² values below the median training performance are expected

## Limitations

- C-band SAR cannot well characterize the understorey due to weak permeability through overstorey layers, limiting predictions for understorey-driven biodiversity measures
- External validation was only demonstrated for two taxa (birds and saproxylic beetles); radar performance dropped for carabid, spider and bat assemblage composition after accounting for regions, reflecting unmeasured regional determinants
- The study regions represent only the major temperate forest ecosystems of Central Europe; generality in other forest types is untested, and georeferenced, well-stratified multi-taxa biodiversity data — the main barrier to larger-scale application — are scarce

## Evidence

- [abstract] Radar performs as well as ALS in biodiversity models; median R² over twelve taxa for the first NMDS axes of assemblage composition is 0.51 (ALS) and 0.57 (radar): "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [abstract] External validation demonstrated on birds and saproxylic beetles: "We further demonstrate the promising predictive ability of radar-derived data with external validation based on the species composition of birds and saproxylic beetles"
- [readme] R packages required for the R-side summary statistics feeding model predictors: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [supplementary] Radar processing basis for gamma_0 predictors used in validation: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
