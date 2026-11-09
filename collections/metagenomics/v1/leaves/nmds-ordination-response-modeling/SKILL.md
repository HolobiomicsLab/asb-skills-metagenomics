---
name: nmds-ordination-response-modeling
description: Use when you have plot-level (1-ha) remote-sensing predictor tables (Sentinel-1
  gamma_0 means or ALS metrics) plus ground-truth biodiversity responses (NMDS axes,
  richness, phylogenetic diversity) for multiple taxa and need to quantify and compare
  predictive power across predictor sets under.
when_to_use_negative:
- Input is raw Sentinel-1 GRDH imagery rather than a plot-level predictor table —
  the SNAP/GPT preprocessing chain (S3.1–S3.2) must be applied first
- Goal is external validation on independent data collected outside the five training
  regions (e.g. birds, saproxylic beetles), which requires a different validation
  workflow
- Responses for groups strongly driven by understorey development — C-band radar is
  expected to underperform for species richness and phylogenetic diversity of such
  groups
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Language in which the boosted GAM fitting, region-grouped five-fold cross-validation,
    and R² summarization are performed (with mboost/gamboost)
- name: raster
  role: R package used to derive pixel- and neighborhood-based summary statistics
    (e.g. S1_mean.csv) that serve as the model input predictor table
- name: glcm
  role: R package required for neighborhood-based texture statistics computed on gamma_0
    temporal stacks before model fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:9ceecb35d11de9ce0f0567460c5bf5629675ba38f827e7ec92afa0af8f6f458e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glcm
  - raster
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmds-ordination-response-modeling

## Summary

Fit boosted generalized additive models (R package mboost/gamboost) that predict NMDS assemblage-composition axis scores, log-transformed species richness, and phylogenetic diversity for 12 functional groups from either Sentinel-1 gamma_0 time-series statistics or ALS structural metrics, and compare predictor-set performance via region-grouped five-fold cross-validated out-of-fold R².

## When to use

Use when you have plot-level (1-ha) remote-sensing predictor tables (Sentinel-1 gamma_0 means or ALS metrics) plus ground-truth biodiversity responses (NMDS axes, richness, phylogenetic diversity) for multiple taxa and need to quantify and compare predictive power across predictor sets under spatially grouped cross-validation.

## When NOT to use

- Input is raw Sentinel-1 GRDH imagery rather than a plot-level predictor table — the SNAP/GPT preprocessing chain (S3.1–S3.2) must be applied first
- Goal is external validation on independent data collected outside the five training regions (e.g. birds, saproxylic beetles), which requires a different validation workflow
- Responses for groups strongly driven by understorey development — C-band radar is expected to underperform for species richness and phylogenetic diversity of such groups

## Inputs

- Plot-level (1-ha) Sentinel-1 gamma_0 time-series summary statistics, e.g. S1_mean.csv (means over VH and VV temporal stacks)
- ALS structural predictor table for the same 1-ha plots
- NMDS axis 1 and axis 2 scores of assemblage composition per functional group (12 groups)
- Log-transformed species richness per functional group
- Phylogenetic diversity per functional group
- Study-region identifiers per plot (used to define region-grouped CV folds)

## Outputs

- boosted_gam_r2_summary.csv containing per-functional-group and median out-of-fold R² for each response and predictor set
- Median R² comparison across 12 functional groups for ALS vs radar (e.g. NMDS1: 0.51 vs 0.57)

## How to apply

Load the radar predictor table of plot-level means of Sentinel-1 gamma_0 time-series statistics (e.g. S1_mean.csv, derived from VH/VV temporal stacks) and the equivalent ALS structural predictor table for the 1-ha plots. Assemble ground-truth responses for each of the 12 functional groups: NMDS axis 1 and 2 scores (assemblage composition), log-transformed species richness, and phylogenetic diversity. Fit boosted GAMs with gamboost for each response × group × predictor set, using five-fold cross-validation in which folds are grouped by study region to prevent spatial leakage across the five temperate forest regions. Compute out-of-fold R² per functional group and response, then take the median R² across the 12 groups per response and predictor set (ALS vs radar). This directly tests whether radar sensitivity to habitat structure matches ALS, the current gold standard for forest structure measurement.

## Related tools

- **R** (Language in which the boosted GAM fitting, region-grouped five-fold cross-validation, and R² summarization are performed (with mboost/gamboost))
- **raster** (R package used to derive pixel- and neighborhood-based summary statistics (e.g. S1_mean.csv) that serve as the model input predictor table)
- **glcm** (R package required for neighborhood-based texture statistics computed on gamma_0 temporal stacks before model fitting)

## Examples

```
Rscript -e 'library(mboost); fit <- gamboost(NMDS1 ~ ., data = S1_mean); cvfit <- cvrisk(fit, folds = region_grouped_5fold)'
```

## Evaluation signals

- Computed median R² over the 12 functional groups must reproduce reported values: NMDS1 0.51 (ALS) vs 0.57 (radar), NMDS2 0.30 vs 0.27, species richness 0.21 vs 0.11, phylogenetic diversity 0.19 vs 0.16
- Cross-validation folds must be grouped by study region (no plots from the same region in both training and test folds)
- boosted_gam_r2_summary.csv must contain per-group R² for all 12 functional groups × 4 responses × 2 predictor sets before median aggregation
- Out-of-fold R² values must be derived only from held-out fold predictions, not in-sample fitted values
- Radar and ALS performance should be broadly comparable, confirming radar performs as well as ALS in biodiversity models

## Limitations

- C-band Sentinel-1 data underpredict species richness and phylogenetic diversity for groups driven by understorey development, due to weak permeability through overstorey layers at short wavelengths
- Phylogenetic diversity of higher trophic groups was not well predicted, as model determinants were associated with understorey density
- Predictive power drops for carabid, spider and bat assemblage composition after accounting for regions, reflecting unmeasured regional determinants
- The five study regions represent only major temperate forest ecosystems of Central Europe; generality in other forests is untested
- Larger-scale application is limited by the lack of georeferenced, well-stratified biodiversity data spanning multiple taxa

## Evidence

- [abstract] median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [discussion] In biodiversity models of 12 functional groups, radar and ALS performed equally well.: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
- [methods] Like the rest of the manuscript it was performed using R.: "Like the rest of the manuscript it was performed using R."
- [readme] Calculating mean values of all our 1-ha plots: S1_mean.csv: "Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [discussion] Weak permeability through overstorey layers is unavoidable with space-borne C-band SAR systems, due to their short wavelengths: "Weak permeability through overstorey layers is unavoidable with space-borne C-band SAR systems, due to their short wavelengths"
