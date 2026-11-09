---
name: biodiversity-diversity-metric-modelling
description: Use when biodiversity response variables (taxonomic/phylogenetic alpha-
  and beta-diversity) are modelled against ALS or Sentinel-1 radar plot predictors
  and you must quantify the explained-variance loss after accounting for forest region
  as a random effect.
when_to_use_negative:
- Input is a single biodiversity metric or a single predictor set — the skill requires
  paired ALS and radar predictor tables over 48 response variables to compare median
  R-squared losses
- Goal is Sentinel-1 SAR data pre-processing (GRDH to gamma_0 via the SNAP/GPT batch
  file Sentinel1_GRDH_to_gamma0.xml) — use the pre-processing workflow instead of
  this modelling skill
- Plots are not spatially replicated across the five forest regions, so a region random-intercept
  factor cannot be defined
edam_operation: http://edamontology.org/operation_2945
tools:
- name: R
  role: Environment for fitting fixed-effects and region-random-intercept mixed-effects
    models of the 48 biodiversity response variables against ALS and radar predictors
    and computing R-squared
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:6b64e67a6c2406ec3d3f36944f21dc0150272f76cc6cf84d57aac681fa440ce8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# biodiversity-diversity-metric-modelling

## Summary

Models ground-truth taxonomic and phylogenetic biodiversity metrics (alpha- and beta-diversity across 12 functional groups, 48 response variables) with ALS versus Sentinel-1 radar per-plot predictors in R, and quantifies how much explained variance (R-squared) is lost when forest region is added as a random intercept factor.

## When to use

Use when biodiversity response variables (taxonomic/phylogenetic alpha- and beta-diversity) are modelled against ALS or Sentinel-1 radar plot predictors and you must quantify the explained-variance loss after accounting for forest region as a random effect.

## When NOT to use

- Input is a single biodiversity metric or a single predictor set — the skill requires paired ALS and radar predictor tables over 48 response variables to compare median R-squared losses
- Goal is Sentinel-1 SAR data pre-processing (GRDH to gamma_0 via the SNAP/GPT batch file Sentinel1_GRDH_to_gamma0.xml) — use the pre-processing workflow instead of this modelling skill
- Plots are not spatially replicated across the five forest regions, so a region random-intercept factor cannot be defined

## Inputs

- 48 biodiversity response variables (taxonomic and phylogenetic alpha- and beta-diversity measures across 12 functional groups, e.g. first NMDS axes of assemblage composition)
- ALS (airborne laser scanning) per-plot predictor table
- Sentinel-1 radar per-plot predictor table, including radar plot means in S1_mean.csv
- Region labels assigning each plot to one of the five temperate forest regions

## Outputs

- mixed_effects_model_results.csv containing per-model R-squared for fixed-effects and mixed-effects fits, per-model explained-variance loss, and median loss per predictor set (ALS vs radar)
- Median explained-variance loss for ALS models and radar models across the 48 response variables

## How to apply

Load the 48 biodiversity response variables (taxonomic and phylogenetic alpha- and beta-diversity measures across 12 functional groups) together with the ALS and Sentinel-1 radar per-plot predictor tables, including the radar plot means provided in S1_mean.csv, into R. For each response variable, fit a fixed-effects model separately with the ALS predictors and with the radar predictors. Refit each model as a mixed-effects model with region (the five temperate forest regions) included as a random intercept factor. Compute the explained variance (R-squared) for each fixed-effects fit and its paired mixed-effects fit, and derive the per-model loss of explained variance as their difference; a random-intercept-only model cannot increase R-squared, so the loss must be non-negative. Aggregate the per-model losses across all 48 response variables and report the median loss separately for ALS models and radar models, comparing against the reported reference medians of 0.14 (ALS) and 0.10 (radar). Write the full per-model results table to mixed_effects_model_results.csv.

## Related tools

- **R** (Environment for fitting fixed-effects and region-random-intercept mixed-effects models of the 48 biodiversity response variables against ALS and radar predictors and computing R-squared) — https://cran.r-project.org/

## Evaluation signals

- Reproduced median explained-variance loss across the 48 response variables is approximately 0.14 for ALS models and 0.10 for radar models
- Each of the 48 response variables yields four fitted models (fixed and mixed x ALS and radar) with non-missing R-squared values
- Every per-model loss (fixed-effects R-squared minus mixed-effects R-squared) is >= 0, since adding region as a random intercept cannot increase explained variance
- mixed_effects_model_results.csv exists and contains one row per response variable with both predictor-set columns populated
- Per-taxa check: R-squared values are consistent with reported medians (e.g. median R-squared over twelve taxa of 0.51 for ALS and 0.57 for radar for first NMDS assemblage-composition axes)

## Limitations

- The drop in predictive power for carabid, spider and bat assemblage composition after accounting for regions may reflect geographical patterns or unmeasured environmental determinants — regional confounding is not fully resolved by the random intercept
- C-band Sentinel-1 cannot well characterize the understorey due to weak permeability through overstorey layers, so species richness and phylogenetic diversity of understorey-driven groups are poorly predicted
- The five forest regions represent only the major temperate forest ecosystems of Central Europe; generality to other forests is untested

## Evidence

- [results] median loss of explained variance: [evidence span withheld — anchor status: not_found]
- [readme] R as the analysis language: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [discussion] equal performance of radar and ALS: "In biodiversity models of 12 functional groups, radar and ALS performed equally well."
- [abstract] reported median R-squared benchmarks: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [discussion] regional confounding caveat: "The drop in predictive power regarding the assemblage composition of carabids, spiders and bats after accounting for regions may reflect the geographical patterns or unmeasured environmental"
