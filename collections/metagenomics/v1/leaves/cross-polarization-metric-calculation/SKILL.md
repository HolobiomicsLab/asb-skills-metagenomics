---
name: cross-polarization-metric-calculation
description: Use when terrain-corrected Sentinel-1 gamma_0 stacks for VV and VH plus
  an acquisition-dates table (e.g., ALB_acquisition_dates.csv) are available and per-pixel
  cross-polarization difference/ratio rasters are needed as radar habitat-structure
  predictors.
when_to_use_negative:
- Inputs are uncalibrated GRDH digital numbers or beta_0 - metrics require gamma_0
  after radiometric calibration, radiometric terrain flattening (SRTM 1 sec DEM),
  and range-doppler terrain correction, otherwise values are incomparable across acquisitions.
- Only a single acquisition is available - speckle handling here relies on medians
  across a multi-date Sentinel-1 time series, not spatial speckle filters.
- The target is understorey structure or understorey-driven biodiversity - C-band
  SAR's weak permeability through overstorey layers limits these backscatter metrics.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: R
  role: Runtime for sourcing Sentinel1_pixel_neighborhood_statistics.r, which loads
    the gamma_0 stacks and computes the cross-polarization summary statistics
  repo: https://cran.r-project.org/
- name: raster
  role: Reads the ALB_gamma0_VH.tif / ALB_gamma0_VV.tif temporal stacks, assigns acquisitions
    to seasonal subsets via ALB_acquisition_dates.csv, and performs per-pixel medians
    and band math (VV-VH, VV/VH)
- name: rgdal
  role: Reads/writes the GeoTIFF inputs and writes the pixel_summary_statistics.tif
    output through GDAL drivers
- name: SNAP toolbox
  role: 'Upstream producer of the inputs: apply precise orbit, remove border/thermal
    noise, radiometric calibration to beta_0, radiometric terrain flattening to gamma_0
    (SRTM 1 sec DEM), and range-doppler terrain correction at 10 m'
  repo: https://step.esa.int/main/download/snap-download/
- name: Sentinel1_pixel_neighborhood_statistics.r
  role: Reference R implementation of the pixel-based summary statistics including
    the cross-polarization metrics, distributed with the paper
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:88620f66a70c9c79e4af810e76be5c2dc4f68c7f35cbbdf295a8c62d67d9ef15
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SNAP toolbox
  - Sentinel1_pixel_neighborhood_statistics.r
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-polarization-metric-calculation

## Summary

Computes per-pixel cross-polarization summary statistics - the VV-VH difference, the VV/VH ratio, and winter-minus-summer differences - from temporal stacks of Sentinel-1 gamma_0 backscatter, yielding 10 m habitat-structure predictor rasters for forest biodiversity models.

## When to use

Use when terrain-corrected Sentinel-1 gamma_0 stacks for VV and VH plus an acquisition-dates table (e.g., ALB_acquisition_dates.csv) are available and per-pixel cross-polarization difference/ratio rasters are needed as radar habitat-structure predictors.

## When NOT to use

- Inputs are uncalibrated GRDH digital numbers or beta_0 - metrics require gamma_0 after radiometric calibration, radiometric terrain flattening (SRTM 1 sec DEM), and range-doppler terrain correction, otherwise values are incomparable across acquisitions.
- Only a single acquisition is available - speckle handling here relies on medians across a multi-date Sentinel-1 time series, not spatial speckle filters.
- The target is understorey structure or understorey-driven biodiversity - C-band SAR's weak permeability through overstorey layers limits these backscatter metrics.

## Inputs

- Temporal stack of VH gamma_0 rasters as GeoTIFF (e.g., ALB_gamma0_VH.tif)
- Temporal stack of VV gamma_0 rasters as GeoTIFF (e.g., ALB_gamma0_VV.tif)
- Sentinel-1 acquisition dates table (ALB_acquisition_dates.csv)
- Plot-center shapefile for extraction/validation (ALB_example_plots.*)
- Reference 1-ha plot means for validation (S1_mean.csv)

## Outputs

- Per-pixel VV-VH difference raster
- Per-pixel VV/VH ratio raster
- Winter-minus-summer difference rasters for VV and VH
- GeoTIFF stack of pixel summary statistics (pixel_summary_statistics.tif) grid-aligned to the 10 m input gamma_0 grid

## How to apply

Load the temporal stacks of VH and VV gamma_0 (ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) plus ALB_acquisition_dates.csv into R with the raster package; the stacks must sit on the 10 m geographic grid produced by range-doppler terrain correction in SNAP. Use the acquisition dates to assign each acquisition to yearly, summer, and winter subsets and compute the per-pixel median backscatter for each polarization and subset, so speckle is smoothed implicitly in time rather than by spatial filtering that would cost geometric detail. Derive the cross-polarization metrics per pixel from these medians: the VV-VH difference, the VV/VH ratio, and the winter-minus-summer difference for each polarization. Write the summary statistic rasters to pixel_summary_statistics.tif with the raster/rgdal GeoTIFF drivers. Validate that the outputs are grid-aligned with the input gamma_0 stacks and that plot-level means extracted with the ALB_example_plots shapefile match the reference means in S1_mean.csv.

## Related tools

- **R** (Runtime for sourcing Sentinel1_pixel_neighborhood_statistics.r, which loads the gamma_0 stacks and computes the cross-polarization summary statistics) — https://cran.r-project.org/
- **raster** (Reads the ALB_gamma0_VH.tif / ALB_gamma0_VV.tif temporal stacks, assigns acquisitions to seasonal subsets via ALB_acquisition_dates.csv, and performs per-pixel medians and band math (VV-VH, VV/VH))
- **rgdal** (Reads/writes the GeoTIFF inputs and writes the pixel_summary_statistics.tif output through GDAL drivers)
- **SNAP toolbox** (Upstream producer of the inputs: apply precise orbit, remove border/thermal noise, radiometric calibration to beta_0, radiometric terrain flattening to gamma_0 (SRTM 1 sec DEM), and range-doppler terrain correction at 10 m) — https://step.esa.int/main/download/snap-download/
- **Sentinel1_pixel_neighborhood_statistics.r** (Reference R implementation of the pixel-based summary statistics including the cross-polarization metrics, distributed with the paper) — https://github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
source("Sentinel1_pixel_neighborhood_statistics.r")
```

## Evaluation signals

- Derived rasters share extent, CRS, and the 10 m pixel spacing of the terrain-corrected input gamma_0 stacks (grid-alignment check).
- Plot-level means extracted from the new statistic rasters are consistent with the reference values in S1_mean.csv.
- pixel_summary_statistics.tif contains bands for both the VV-VH difference and the VV/VH ratio alongside the seasonal medians.
- No spatial speckle filter was applied: outputs retain full 10 m geometric detail, with speckle reduced only by temporal medians over acquisitions.
- Spot-check a pixel's median against a manual subset of the stack built from the dates in ALB_acquisition_dates.csv.

## Limitations

- C-band SAR's weak permeability through overstorey layers limits sensitivity to the understorey, so these metrics under-predict species richness and phylogenetic diversity of understorey-driven groups.
- Speckle suppression depends entirely on temporal medians over a dense multi-acquisition Sentinel-1 time series; single-scene or sparse inputs will be speckle-noisy.
- Metrics inherit artifacts from upstream preprocessing - skipped border-noise, thermal-noise removal, or terrain flattening corrupts VV/VH comparability across acquisitions.

## Evidence

- [other] Per-pixel cross-polarization metrics are the VV-VH difference, the VV/VH ratio, and winter-minus-summer differences per polarization.: [evidence span withheld — anchor status: not_found]
- [methods] Summary statistics are computed in R on temporal stacks of Sentinel-1 gamma_0 data.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Speckle is smoothed implicitly by temporal medians rather than spatial filtering, preserving geometric detail.: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in space, thus not causing any further loss of geometric detail"
- [readme] Required R packages are rgeos, raster, rgdal, and glcm.: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Input gamma_0 stacks are in geographic coordinates with 10 m pixel spacing.: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [readme] S1_mean.csv provides reference 1-ha plot means used to validate the derived statistics.: "Calculated mean values of all our 1-ha plots: S1_mean.csv"
