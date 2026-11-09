---
name: als-radar-predictor-comparison
description: Use when a study provides two alternative remote-sensing predictor sets
  (e.g. ALS and Sentinel-1 radar) for the same plot-level biodiversity responses and
  you must quantify and validate their relative predictive power.
when_to_use_negative:
- The goal is Sentinel-1 GRDH-to-gamma_0 preprocessing itself — use the SNAP/GPT batch
  workflow (Sentinel1_GRDH_to_gamma0.xml) instead; this skill starts from an already-derived
  predictor table.
- Only one predictor set is available (radar-only or ALS-only), so no ALS-vs-radar
  comparison or mixed-effects region comparison is possible.
- Responses are not plot-level biodiversity measures (composition axes, richness,
  phylogenetic diversity) — e.g. direct habitat-structure comparison between radar
  and ALS point clouds is a different skill.
tools:
- name: R
  role: Fitting boosted GAMs and mixed-effects models with region as random factor,
    computing cross-validated R², and deriving the pixel/neighborhood summary statistics
    from gamma_0 stacks (with packages rgeos, raster, rgdal, glcm)
- name: Graph Processing Tools (GPT)
  role: Batch processing of Sentinel-1 GRDH scenes to gamma_0 via the provided Sentinel1_GRDH_to_gamma0.xml
    chain, upstream of predictor-table construction
- name: SNAP toolbox
  role: Hosts the GPT command-line interface used to run the radar preprocessing chain
    (orbit correction, border/thermal noise removal, calibration, terrain flattening,
    terrain correction)
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:515cc8684a96737f459459429f158baa09be864ab66ead116f6d5e0b312fb882
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Graph Processing Tools (GPT)
  - R
  - SNAP toolbox
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# als-radar-predictor-comparison

## Summary

Head-to-head comparison of ALS structural predictors vs Sentinel-1 gamma_0 time-series radar predictors for modelling ground-truth biodiversity (NMDS composition axes, species richness, phylogenetic diversity) of 12 functional groups, including region-aware validation and mixed-effects sensitivity analysis.

## When to use

Use when a study provides two alternative remote-sensing predictor sets (e.g. ALS and Sentinel-1 radar) for the same plot-level biodiversity responses and you must quantify and validate their relative predictive power.

## When NOT to use

- The goal is Sentinel-1 GRDH-to-gamma_0 preprocessing itself — use the SNAP/GPT batch workflow (Sentinel1_GRDH_to_gamma0.xml) instead; this skill starts from an already-derived predictor table.
- Only one predictor set is available (radar-only or ALS-only), so no ALS-vs-radar comparison or mixed-effects region comparison is possible.
- Responses are not plot-level biodiversity measures (composition axes, richness, phylogenetic diversity) — e.g. direct habitat-structure comparison between radar and ALS point clouds is a different skill.

## Inputs

- Radar predictor table: plot-level means of Sentinel-1 gamma_0 time-series statistics (S1_mean.csv) for 1-ha plots
- ALS structural predictor table for the same 1-ha plots
- Ground-truth biodiversity responses for 12 functional groups: NMDS axis 1 and 2 scores, log-transformed species richness, phylogenetic diversity (48 alpha- and beta-diversity variables in total)
- Study-region assignment (five temperate forest regions in central Europe) for region-grouped CV folds and random intercepts

## Outputs

- Per-group and median out-of-fold R² values for each response and predictor set (boosted_gam_r2_summary.csv)
- Fixed-effects vs mixed-effects R² and per-model explained-variance loss for all 48 response variables (mixed_effects_model_results.csv)
- Median R² medians for ALS vs radar, e.g. NMDS1: 0.51 (ALS) vs 0.57 (radar)

## How to apply

Load the radar predictor table (plot-level means of Sentinel-1 gamma_0 time-series statistics, e.g. S1_mean.csv) and the ALS structural predictor table for the 1-ha plots, together with the ground-truth biodiversity responses: NMDS axis 1 and 2 scores (assemblage composition), log-transformed species richness, and phylogenetic diversity for the 12 functional groups. Fit boosted GAMs (e.g. mboost/gamboost in R) separately for each predictor set and response, using five-fold cross-validation in which folds are grouped by the five study regions to prevent spatial leakage, then compute out-of-fold R² per functional group and take the median across the 12 groups. As a sensitivity check, refit each of the 48 taxonomic/phylogenetic alpha- and beta-diversity models as a mixed-effects model with region as a random intercept and compute the loss of explained variance (R²) relative to the fixed-effects fit. Rationale: since median backscatter over multiple acquisitions smooths speckle implicitly in time rather than in space, the radar stack (VH and VV gamma_0 at 10 m) can be treated as a structural predictor set directly comparable to ALS without additional spatial filtering.

## Related tools

- **R** (Fitting boosted GAMs and mixed-effects models with region as random factor, computing cross-validated R², and deriving the pixel/neighborhood summary statistics from gamma_0 stacks (with packages rgeos, raster, rgdal, glcm))
- **Graph Processing Tools (GPT)** (Batch processing of Sentinel-1 GRDH scenes to gamma_0 via the provided Sentinel1_GRDH_to_gamma0.xml chain, upstream of predictor-table construction)
- **SNAP toolbox** (Hosts the GPT command-line interface used to run the radar preprocessing chain (orbit correction, border/thermal noise removal, calibration, terrain flattening, terrain correction))

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r  # sources the README script over ALB_gamma0_VH/VV.tif stacks + ALB_example_plots shapefile to build plot-level S1_mean.csv predictors
```

## Evaluation signals

- Computed median R² over the 12 functional groups must reproduce the reported values: NMDS1 0.51 (ALS) vs 0.57 (radar), NMDS2 0.30 vs 0.27, species richness 0.21 vs 0.11, phylogenetic diversity 0.19 vs 0.16.
- Median explained-variance loss after adding region as random intercept must match 0.14 for ALS models and 0.10 for radar models across the 48 response variables.
- CV folds must be grouped by study region (no plots from the same region split across folds) to prevent spatial leakage; verify fold assignments.
- Output CSVs must contain per-group/per-model R² rows plus the median summaries, allowing the full results table to be checked against reference values.
- Radar and ALS median R² values should be broadly comparable (radar within ~0.06 of ALS for composition axes), consistent with 'radar and ALS performed equally well'.

## Limitations

- C-band SAR cannot well characterize the understorey due to weak permeability through overstorey layers, so radar underpredicts species richness and phylogenetic diversity for groups driven by understorey development (e.g. higher trophic groups).
- After accounting for regions, predictive power drops for the assemblage composition of carabids, spiders and bats, reflecting geographical patterns or unmeasured environmental determinants.
- The five study regions represent only major temperate forest ecosystems of Central Europe; generality in other forest types is untested.
- Radar models for species richness (0.11) and phylogenetic diversity (0.16) perform clearly below ALS for some responses, so equal performance does not hold for every biodiversity facet.

## Evidence

- [abstract] median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [abstract] We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard in the measurement of forest structure: "We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard in the measurement of forest structure"
- [discussion] In biodiversity models of 12 functional groups, radar and ALS performed equally well.: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
- [methods] This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing.: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [readme] due to the approach taken in this manuscript, that is, calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time: "due to the approach taken in this manuscript, that is, calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time"
