---
name: sar-gamma0-plot-summary-statistics
description: Use when you have Sentinel-1 gamma_0 temporal stacks (VH/VV GeoTIFFs),
  acquisition dates, and plot-center shapefiles and need per-plot radar structural
  metrics as predictors for ecological models (CCoA, NMDS composition models).
when_to_use_negative:
- Input is raw GRDH digital-number imagery — it must first be converted to gamma_0
  via the SNAP batch chain (orbit file, border/thermal noise removal, calibration,
  terrain flattening, range-doppler terrain correction) before summary statistics
  are meaningful.
- 'Analysis requires understorey-sensitive metrics: C-band SAR has weak permeability
  through overstorey layers, so species richness and phylogenetic diversity of understorey-driven
  groups are poorly predicted from these radar statistics.'
- Spatial speckle smoothing is explicitly desired — this workflow smooths speckle
  implicitly in time via multi-acquisition medians instead, and adding spatial filtering
  would cause further loss of geometric detail.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: R
  role: Environment for the pixel- and neighborhood-based summary statistics script
    (Sentinel1_pixel_neighborhood_statistics.r)
  repo: https://cran.r-project.org/
- name: raster
  role: Read and manipulate the VH/VV gamma_0 temporal stacks and extract per-plot
    statistics
- name: rgdal
  role: Read the plot-center shapefile and handle geolocation of stacks
- name: rgeos
  role: Geometric operations for buffer/plot-footprint handling around plot centers
- name: glcm
  role: Compute neighborhood-based GLCM texture statistics from the gamma_0 stacks
- name: SNAP toolbox / GPT
  role: Upstream batch (Sentinel1_GRDH_to_gamma0.xml) producing the gamma_0 stacks
    that this skill consumes
  repo: https://step.esa.int/main/download/snap-download/
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:b182d82f5c91796964d49edefd82c9b050d9470b215c7b976b992513adb80918
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# sar-gamma0-plot-summary-statistics

## Summary

Compute pixel- and neighborhood-based summary statistics (e.g., mean backscatter and GLCM texture) from temporal stacks of Sentinel-1 VH/VV gamma_0 data for fixed plot areas (1-ha), producing the radar predictor table (S1_mean.csv) used in biodiversity models. Median compositing across acquisitions suppresses speckle implicitly in time rather than in space.

## When to use

Use when you have Sentinel-1 gamma_0 temporal stacks (VH/VV GeoTIFFs), acquisition dates, and plot-center shapefiles and need per-plot radar structural metrics as predictors for ecological models (CCoA, NMDS composition models).

## When NOT to use

- Input is raw GRDH digital-number imagery — it must first be converted to gamma_0 via the SNAP batch chain (orbit file, border/thermal noise removal, calibration, terrain flattening, range-doppler terrain correction) before summary statistics are meaningful.
- Analysis requires understorey-sensitive metrics: C-band SAR has weak permeability through overstorey layers, so species richness and phylogenetic diversity of understorey-driven groups are poorly predicted from these radar statistics.
- Spatial speckle smoothing is explicitly desired — this workflow smooths speckle implicitly in time via multi-acquisition medians instead, and adding spatial filtering would cause further loss of geometric detail.

## Inputs

- Temporal stacks of VH and VV Sentinel-1 gamma_0 GeoTIFFs (e.g., ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) at 10 m pixel spacing
- Sentinel-1 acquisition dates CSV (e.g., ALB_acquisition_dates.csv)
- Shapefile of plot centers (e.g., ALB_example_plots.*) defining 1-ha plot footprints
- SRTM 1 sec DEM (used upstream for terrain flattening and orthorectification)

## Outputs

- Per-plot mean backscatter summary statistics table (S1_mean.csv) for all 1-ha plots
- Pixel- and neighborhood-based (GLCM) texture statistics per plot

## How to apply

Start from calibrated, terrain-flattened, orthorectified gamma_0 stacks (10 m pixels) produced by the SNAP/GPT chain (Apply precise orbit, Remove border noise, Remove thermal noise, Radiometric calibration, Radiometric terrain flattening, Range-doppler terrain correction). In R, load the VH and VV temporal stacks, the acquisition-dates CSV, and the plot-center shapefile (minimum packages: rgeos, raster, rgdal, glcm) and source Sentinel1_pixel_neighborhood_statistics.r to extract per-pixel and neighborhood-based summary statistics within each plot footprint. Median backscatter over multiple acquisitions smooths speckle in time, so no additional spatial speckle filter is applied, preserving geometric detail. Compute plot-mean values for all 1-ha plots and write them to S1_mean.csv; this table feeds downstream steps such as CCoA against ALS metrics and external validation of NMDS1 assemblage-composition models. Supplement with GLCM texture metrics where neighborhood structure is needed.

## Related tools

- **R** (Environment for the pixel- and neighborhood-based summary statistics script (Sentinel1_pixel_neighborhood_statistics.r)) — https://cran.r-project.org/
- **raster** (Read and manipulate the VH/VV gamma_0 temporal stacks and extract per-plot statistics)
- **rgdal** (Read the plot-center shapefile and handle geolocation of stacks)
- **rgeos** (Geometric operations for buffer/plot-footprint handling around plot centers)
- **glcm** (Compute neighborhood-based GLCM texture statistics from the gamma_0 stacks)
- **SNAP toolbox / GPT** (Upstream batch (Sentinel1_GRDH_to_gamma0.xml) producing the gamma_0 stacks that this skill consumes) — https://step.esa.int/main/download/snap-download/

## Examples

```
source("Sentinel1_pixel_neighborhood_statistics.r")  # after setting input dir with ALB_gamma0_VH/VV.tif, ALB_acquisition_dates.csv, ALB_example_plots.*
```

## Evaluation signals

- S1_mean.csv contains one row per 1-ha plot with mean VH and VV gamma_0 summary statistics and no missing values for plots present in the shapefile.
- Output values are in gamma_0 backscatter units (not raw DN or beta_0), confirming the upstream SNAP chain ran before statistics were computed.
- Plugging the resulting radar predictors into the article's fixed-effects models reproduces the reported performance: median R² of 0.57 for first NMDS axes over twelve taxa (vs. 0.51 for ALS) and external-validation R² ≈ 0.26 (birds, n=72) and ≈ 0.22 (saproxylic beetles, n=91).
- In CCoA against ALS metrics, the radar metric set yields a first canonical correlation ≈ 0.92 and nine significant axis pairs explaining ~96.30% of variance — indicating the statistics capture habitat structure comparably to ALS.
- No separate spatial speckle filter was applied; speckle smoothing is achieved by medians over the multi-acquisition temporal stack.

## Limitations

- C-band SAR weakly penetrates overstorey layers, so understorey-driven richness and phylogenetic diversity are poorly captured.
- Predictive power drops for carabid, spider, and bat assemblage composition after accounting for regions, suggesting unmeasured regional determinants.
- The five study regions represent only major temperate forest ecosystems of Central Europe; generality elsewhere is untested (though external validation showed R² of 0.26 for birds and 0.22 for beetles outside training regions).
- Precise orbit data are only available ~2 weeks after data take, constraining near-real-time processing.

## Evidence

- [methods] Derivation of pixel- and neighborhood-based summary statistics: "Derivation of pixel- and neighborhood-based summary statistics"
- [readme] R packages required: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Implicit temporal speckle smoothing: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in the space"
- [readme] gamma_0 output specification: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [readme] Example input files for the R script: "Two temporal stacks of VH and VV gamma_0 data: 
     ALB_gamma0_VH.tif and ALB_gamma0_VV.tif"
