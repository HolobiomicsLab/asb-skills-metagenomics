---
name: biodiversity-alpha-beta-phylogenetic-metrics
description: Use when plot-level taxon occurrence/abundance data across multiple trophic
  groups must be converted into α, β (NMDS composition axes), and phylogenetic diversity
  response tables for modelling against remote-sensing predictors.
when_to_use_negative:
- Input is already a biodiversity response table (e.g. precomputed NMDS scores or
  S1_mean.csv predictor tables) rather than raw species observations
- The goal is Sentinel-1 SAR pre-processing (orbit correction, thermal noise removal,
  radiometric terrain flattening) rather than biodiversity metric derivation
- Taxa lack phylogenetic information or stratified georeferenced sampling — the article
  notes the lack of well-stratified, georeferenced multi-taxon biodiversity data is
  the major barrier to larger-scale application
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: R
  role: Environment for ordination, phylogenetic diversity computation, and boosted
    GAM (gamboost/mboost) modelling with region-grouped five-fold cross-validation
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:a37ce279e48d894512ed3a796ab3099467f5e2cf32bcf1cea334311343849e94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# biodiversity-alpha-beta-phylogenetic-metrics

## Summary

Compute ground-truth biodiversity responses — α-diversity (log-transformed species richness), β-diversity (NMDS axis 1 and 2 scores of assemblage composition), and phylogenetic diversity — per plot for each of 12 functional groups/taxa, for use as response variables in biodiversity–remote-sensing models.

## When to use

Use when plot-level taxon occurrence/abundance data across multiple trophic groups must be converted into α, β (NMDS composition axes), and phylogenetic diversity response tables for modelling against remote-sensing predictors.

## When NOT to use

- Input is already a biodiversity response table (e.g. precomputed NMDS scores or S1_mean.csv predictor tables) rather than raw species observations
- The goal is Sentinel-1 SAR pre-processing (orbit correction, thermal noise removal, radiometric terrain flattening) rather than biodiversity metric derivation
- Taxa lack phylogenetic information or stratified georeferenced sampling — the article notes the lack of well-stratified, georeferenced multi-taxon biodiversity data is the major barrier to larger-scale application

## Inputs

- Plot-level species occurrence/abundance tables for 12 functional groups (taxa across trophic levels) from five temperate forest regions in central Europe
- Phylogenetic trees per functional group for phylogenetic diversity computation
- Plot polygon/centroid shapefiles defining the 1-ha sampling plots
- External validation species-composition data for birds and saproxylic beetles collected outside the five training regions

## Outputs

- Per-group, per-plot response table of NMDS axis 1 and axis 2 scores (assemblage composition / β-diversity)
- Per-group, per-plot log-transformed species richness (α-diversity) values
- Per-group, per-plot phylogenetic diversity values
- boosted_gam_r2_summary.csv with per-group and median out-of-fold R² per response and predictor set (radar vs ALS)

## How to apply

For each of the 12 functional groups, aggregate species observations to the 1-ha plot level, log-transform species richness for the α-diversity measure, and ordinate plot-by-species matrices with non-metric multidimensional scaling, retaining the first two NMDS axes to represent assemblage composition (β-diversity). Compute phylogenetic diversity per plot from the group's phylogeny, noting that this measure tracks evolutionary distinctness of the local assemblage. These responses (NMDS1, NMDS2, log richness, phylogenetic diversity) are then modelled per functional group with boosted GAMs against Sentinel-1 gamma_0 radar or ALS structural predictors, with five-fold cross-validation whose folds are grouped by study region to prevent spatial leakage. Judge success by reproducing the reported median out-of-fold R² across the 12 groups: NMDS1 0.51 (ALS) vs 0.57 (radar), NMDS2 0.30 vs 0.27, species richness 0.21 vs 0.11, phylogenetic diversity 0.19 vs 0.16.

## Related tools

- **R** (Environment for ordination, phylogenetic diversity computation, and boosted GAM (gamboost/mboost) modelling with region-grouped five-fold cross-validation) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r  # then in R: library(mboost); cvfit <- gamboost(y ~ ., data = model_df, control = boost_control(mstop = cv(CVfolds, group = region)))
```

## Evaluation signals

- Median out-of-fold R² across 12 functional groups reproduces reported values: NMDS1 0.51 (ALS) vs 0.57 (radar); NMDS2 0.30 vs 0.27; species richness 0.21 vs 0.11; phylogenetic diversity 0.19 vs 0.16
- Cross-validation folds are grouped by study region, so no plots from the same region appear in both training and test folds (no spatial leakage)
- boosted_gam_r2_summary.csv contains per-group R² rows for all 12 functional groups × 4 responses × 2 predictor sets
- NMDS solutions are stable across runs and axis 1/2 scores cover all 1-ha plots in each region
- Radar and ALS model performance is statistically indistinguishable for assemblage composition, confirming the 'performed equally well' finding

## Limitations

- Sentinel-1 (C-band SAR) poorly predicts species richness and phylogenetic diversity for groups driven by understorey development, due to weak permeability through overstorey layers at short wavelengths
- Phylogenetic diversity of higher trophic groups was not well predicted because model determinants were associated with understorey density
- Predictive power drops for carabid, spider and bat assemblage composition after accounting for regions, reflecting unmeasured regional determinants
- Study regions represent only the major temperate forest ecosystems of central Europe; generality in other forest types is untested

## Evidence

- [abstract] Median R² of assemblage composition models: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [abstract] Study design scope: "a newly launched satellite-borne radar system (Sentinel-1) to map the biodiversity of twelve taxa across five temperate forest regions in central Europe"
- [intro] Modelling objective: "Thus, we quantify the predictive power of radar in modelling different aspects of biodiversity, including species composition and richness and phylogenetic diversity, and compare the results to those"
- [discussion] Equal performance finding: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
- [discussion] Understorey limitation: "The shortcoming of Sentinel-1 data that we uncovered was in the prediction of species richness and phylogenetic diversity for groups that were more strongly driven by the development of the"
- [discussion] Data availability barrier: "The major barrier to the larger-scale application of our methodology is the lack of availability of georeferenced and well-stratified (both spatially and ecologically) biodiversity data that span"
