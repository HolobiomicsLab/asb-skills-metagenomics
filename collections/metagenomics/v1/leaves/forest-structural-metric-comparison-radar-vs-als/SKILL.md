---
name: forest-structural-metric-comparison-radar-vs-als
description: Use when you have plot-mean structural metrics from two remote-sensing
  sources (e.g. VH/VV gamma_0 temporal stacks vs ALS point clouds) and need to test
  whether the cheaper/spaceborne dataset captures the same forest-structure information.
when_to_use_negative:
- Input is already a single feature table — no second metric set exists to correlate
  against, so CCoA is undefined
- Goal is biodiversity modelling of taxa (species composition/richness) rather than
  comparing remote-sensing structural datasets
- Plots cannot be spatially matched 1:1 between radar and ALS sources, since the analysis
  requires paired plot-level metrics
edam_operation: http://edamontology.org/operation_0314
tools:
- name: R
  role: Runs the canonical correlation analysis between radar and ALS metric sets
    on standardised plot-level data
  repo: https://cran.r-project.org/
- name: raster
  role: R package used to extract/summarise Sentinel-1 gamma_0 temporal-stack values
    over the 1-ha plots feeding the radar metric set
- name: rgdal
  role: R package used to read plot shapefiles for spatial overlay of radar metrics
    on plot polygons
- name: rgeos
  role: R package for geometric operations on plot polygons during neighbourhood statistic
    computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:58be54f50afd5226841ed4a037c4e16b25f383d00336316a5fb845db8ffc17f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_0314
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# forest-structural-metric-comparison-radar-vs-als

## Summary

Compares Sentinel-1 radar-derived habitat-structure metrics against airborne laser scanning (ALS) metrics for the same 1-ha plots, using R-based canonical correlation analysis (CCoA) to quantify how strongly the two remote-sensing datasets capture the same structural gradient.

## When to use

Use when you have plot-mean structural metrics from two remote-sensing sources (e.g. VH/VV gamma_0 temporal stacks vs ALS point clouds) and need to test whether the cheaper/spaceborne dataset captures the same forest-structure information.

## When NOT to use

- Input is already a single feature table — no second metric set exists to correlate against, so CCoA is undefined
- Goal is biodiversity modelling of taxa (species composition/richness) rather than comparing remote-sensing structural datasets
- Plots cannot be spatially matched 1:1 between radar and ALS sources, since the analysis requires paired plot-level metrics

## Inputs

- S1_mean.csv: plot-mean structural metrics derived from VH and VV gamma_0 temporal stacks over 1-ha plots
- ALS-derived structural metrics for the same 1-ha plots
- Shapefile of plot centres (e.g. ALB_example_plots.*) for spatial matching

## Outputs

- ccoa_canonical_structure.csv: canonical correlation per axis pair and structure coefficients (loadings) of radar and ALS variables on retained axes
- List of statistically significant canonical axis pairs (p < 0.05) with proportion of variance explained

## How to apply

Load the radar-derived plot-mean metrics (S1_mean.csv, computed from VH/VV gamma_0 temporal stacks over 1-ha plots) and the ALS-derived structural metrics for the same plots into R. Standardise both variable sets to zero mean and unit variance before analysis. Run canonical correlation analysis (cancor/candisc in R) between the radar metric set and the ALS metric set, test each canonical axis pair for significance (p < 0.05), and extract canonical correlations plus structure correlations (loadings) of the original variables on each axis. The rationale is that agreement between radar and ALS axes demonstrates that freely available Sentinel-1 time series approximate the ALS 'gold standard' of forest structure; in the source study, nine significant pairs explained 96.30% of the shared variance.

## Related tools

- **R** (Runs the canonical correlation analysis between radar and ALS metric sets on standardised plot-level data) — https://cran.r-project.org/
- **raster** (R package used to extract/summarise Sentinel-1 gamma_0 temporal-stack values over the 1-ha plots feeding the radar metric set)
- **rgdal** (R package used to read plot shapefiles for spatial overlay of radar metrics on plot polygons)
- **rgeos** (R package for geometric operations on plot polygons during neighbourhood statistic computation)

## Examples

```
cc <- cancor(scale(radar_metrics), scale(als_metrics)); cbind(pair=1:length(cc$cor), canonical_correlation=cc$cor)
```

## Evaluation signals

- First canonical pair correlation should be approximately 0.92 and second approximately 0.75
- Nine significant canonical axis pairs (p < 0.05) should together account for approximately 96.30% of the variance of the two datasets
- Leading canonical axes should align with a forest-maturity gradient followed by structural-heterogeneity gradient interpretations
- Structure coefficients table should include loadings for all 13 radar and ALS variables per retained axis
- Both variable sets must be verified as standardised (zero mean, unit variance) before cancor is called

## Limitations

- Radar sensitivity to understorey structure is limited: weak permeability through overstorey layers is unavoidable with space-borne C-band SAR due to short wavelengths
- Validation against ALS covers temperate forest ecosystems of Central Europe; generalisation to other forest types is untested
- Speckle is smoothed implicitly in time (median over multiple acquisitions) rather than in space, so results depend on having a dense Sentinel-1 time series

## Evidence

- [abstract] Sensitivity of radar to habitat structure matches ALS gold standard: "We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard in the measurement of forest structure"
- [methods] R packages required for the R-based statistical workflow: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Plot-mean radar metrics file used as CCoA input: "4) Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [discussion] C-band limitation relevant to structural metric agreement: "Weak permeability through overstorey layers is unavoidable with space-borne C-band SAR systems, due to their short wavelengths"
