---
name: r-statistical-programming-spatial-data
description: Use when computing plot-mean structural metrics (e.g. over 1-ha plots)
  from Sentinel-1 VH/VV gamma_0 temporal stacks in R, or when statistically comparing
  radar- and ALS-derived habitat structure via canonical correlation analysis.
when_to_use_negative:
- Input is already a feature table of plot-level metrics and no spatial summarisation
  over 1-ha plots is needed — skip the raster/rgdal statistics step and go straight
  to multivariate modelling.
- Raw, uncalibrated Sentinel-1 GRDH digital numbers — these must first be processed
  to gamma_0 via the SNAP/GPT chain (terrain flattening, terrain correction) before
  R-based statistics are valid.
- Assessing understorey-driven biodiversity components (e.g. species richness or phylogenetic
  diversity of groups driven by understorey development), where C-band SAR's weak
  permeability through overstorey layers is a known limitation.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: R
  role: Environment in which all pixel-, neighborhood-based summary statistics and
    the CCoA between radar and ALS metrics are performed
  repo: https://cran.r-project.org/
- name: raster
  role: Minimum required R package for reading Sentinel-1 gamma_0 temporal stacks
    and computing plot-level raster statistics
- name: rgdal
  role: Minimum required R package for reading the plot-center shapefile and handling
    geolocation of 1-ha plots
- name: rgeos
  role: Minimum required R package for geometric operations on plot polygons when
    summarising radar metrics
- name: glcm
  role: Minimum required R package for texture-based neighborhood statistics on gamma_0
    imagery
- name: SNAP toolbox / GPT
  role: Upstream pre-processing of Sentinel-1 GRDH to gamma_0 (precise orbit, noise
    removal, calibration, terrain flattening/correction) feeding the R statistics
  repo: https://step.esa.int/main/download/snap-download/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:2ef95c3dc66ca5e946f9cbf97d82f3ed97ea97c0cacce7cbdb59ab04d0f29a57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SNAP toolbox / GPT
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# r-statistical-programming-spatial-data

## Summary

Apply R-based spatial and multivariate statistical workflows to Sentinel-1 gamma_0 temporal stacks and plot-level biodiversity data, using the raster, rgdal, rgeos, and glcm packages for pixel- and neighborhood-based summary statistics and canonical correlation analysis (CCoA) between radar and ALS habitat metrics.

## When to use

Use when computing plot-mean structural metrics (e.g. over 1-ha plots) from Sentinel-1 VH/VV gamma_0 temporal stacks in R, or when statistically comparing radar- and ALS-derived habitat structure via canonical correlation analysis.

## When NOT to use

- Input is already a feature table of plot-level metrics and no spatial summarisation over 1-ha plots is needed — skip the raster/rgdal statistics step and go straight to multivariate modelling.
- Raw, uncalibrated Sentinel-1 GRDH digital numbers — these must first be processed to gamma_0 via the SNAP/GPT chain (terrain flattening, terrain correction) before R-based statistics are valid.
- Assessing understorey-driven biodiversity components (e.g. species richness or phylogenetic diversity of groups driven by understorey development), where C-band SAR's weak permeability through overstorey layers is a known limitation.

## Inputs

- Temporal stacks of Sentinel-1 VH and VV gamma_0 GeoTIFF data (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif)
- Sentinel-1 acquisition dates CSV (e.g. ALB_acquisition_dates.csv)
- Shapefile of plot centers (e.g. ALB_example_plots.shp and companions)
- ALS-derived structural metrics for the same 1-ha plots
- Radar-derived plot-mean structural metrics (e.g. S1_mean.csv)

## Outputs

- Plot-mean Sentinel-1 structural metrics per 1-ha plot (e.g. S1_mean.csv)
- Canonical correlations and structure coefficients for significant CCoA axis pairs (e.g. ccoa_canonical_structure.csv)
- Significance tests per canonical axis pair (nine pairs retained at p < 0.05)

## How to apply

After Sentinel-1 GRDH data are pre-processed to gamma_0 (via the SNAP/GPT chain: apply precise orbit, remove border and thermal noise, radiometric calibration, radiometric terrain flattening, range-doppler terrain correction), source the provided R script to derive pixel- and neighborhood-based summary statistics from the VH and VV gamma_0 temporal stacks (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) over 1-ha plots defined by a plot-center shapefile, producing plot-mean values like S1_mean.csv. For the CCoA comparison, load radar-derived plot-mean metrics and the corresponding ALS-derived metrics, standardise both variable sets to zero mean and unit variance, then run canonical correlation analysis (cancor/candisc in R), testing each canonical axis pair and retaining the significant pairs (the study found nine significant pairs at p < 0.05, together explaining 96.30% of the variance, with first-pair correlation ~0.92 and second ~0.75). Write canonical correlations and structure coefficients (loadings) to a CSV such as ccoa_canonical_structure.csv and interpret leading axes against forest-maturity and structural-heterogeneity gradients.

## Related tools

- **R** (Environment in which all pixel-, neighborhood-based summary statistics and the CCoA between radar and ALS metrics are performed) — https://cran.r-project.org/
- **raster** (Minimum required R package for reading Sentinel-1 gamma_0 temporal stacks and computing plot-level raster statistics)
- **rgdal** (Minimum required R package for reading the plot-center shapefile and handling geolocation of 1-ha plots)
- **rgeos** (Minimum required R package for geometric operations on plot polygons when summarising radar metrics)
- **glcm** (Minimum required R package for texture-based neighborhood statistics on gamma_0 imagery)
- **SNAP toolbox / GPT** (Upstream pre-processing of Sentinel-1 GRDH to gamma_0 (precise orbit, noise removal, calibration, terrain flattening/correction) feeding the R statistics) — https://step.esa.int/main/download/snap-download/

## Examples

```
Rscript Sentinel1_pixel_ neighborhood _statistics.r
```

## Evaluation signals

- First canonical correlation pair is ~0.92 and the second ~0.75 between radar and ALS metric sets
- Nine significant canonical axis pairs (p < 0.05) together explain ~96.30% of the variance of the two remote-sensing datasets
- Leading canonical axes align with the reported forest-maturity and structural-heterogeneity gradient interpretations
- Plot-mean outputs (S1_mean.csv) match values computed over 1-ha plots from the VH/VV gamma_0 temporal stacks using the provided R script
- Both radar and ALS variable sets are standardised (zero mean, unit variance) before CCoA, and structure coefficients are reported for retained pairs

## Limitations

- C-band SAR (Sentinel-1) has weak permeability through overstorey layers, so understorey-driven biodiversity components (e.g. species richness and phylogenetic diversity of some groups) are poorly predicted
- The workflow was validated only in five temperate forest regions of Central Europe; generality in other forest ecosystems is untested
- Speckle is not explicitly filtered in space — smoothing relies implicitly on temporal medians over multiple acquisitions, which requires a dense time series

## Evidence

- [readme] Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [results] CCoA identified nine statistically significant canonical axis pairs (p < 0.05) between the 13 radar and ALS axes, together explaining 96.30% of the variance: [evidence span withheld — anchor status: not_found]
- [methods] All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox: "All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox by applying the provided processing chain batch file"
