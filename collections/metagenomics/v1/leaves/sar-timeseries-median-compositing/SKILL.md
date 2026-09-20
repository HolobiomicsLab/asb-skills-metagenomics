---
name: sar-timeseries-median-compositing
description: Use when you have temporal stacks of Sentinel-1 gamma_0 VH/VV backscatter
  plus acquisition dates and need ecologically meaningful, speckle-reduced per-pixel
  summary rasters for habitat-structure analysis.
when_to_use_negative:
- Input is still uncalibrated Sentinel-1 GRDH digital numbers rather than processed
  gamma_0 temporal stacks — the SNAP pre-processing chain (orbit correction, noise
  removal, calibration, terrain flattening, terrain correction) must run first
- You need spatial speckle suppression for a single-scene analysis without a multi-temporal
  stack — the temporal median compositing approach requires multiple acquisitions
  and offers no benefit for single-date data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: R
  role: Environment in which the pixel- and neighborhood-based summary statistics
    are computed
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Loading and manipulating the VH/VV gamma_0 temporal stacks and deriving per-pixel
    statistics
- name: rgdal
  role: Writing the derived summary-statistic rasters to GeoTIFF output
- name: glcm
  role: Minimum required R package listed alongside the summary-statistics workflow
- name: rgeos
  role: Minimum required R package for handling plot-center shapefile geometry
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:963c9d53458fdc641bfff96cb33c33e8488118be59015af952b27c37af840bec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/So-YeonBae/Sentinel1-Biodiversity
  tools:
  - R
  - glcm
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# SAR time-series median compositing

## Summary

Computes per-pixel summary statistics (medians, VV-VH differences, VV/VH ratios, winter-minus-summer differences) from temporal stacks of Sentinel-1 gamma_0 backscatter, implicitly smoothing speckle in time rather than space to preserve geometric detail for forest structure mapping.

## When to use

Use when you have temporal stacks of Sentinel-1 gamma_0 VH/VV backscatter plus acquisition dates and need ecologically meaningful, speckle-reduced per-pixel summary rasters for habitat-structure analysis.

## When NOT to use

- Input is still uncalibrated Sentinel-1 GRDH digital numbers rather than processed gamma_0 temporal stacks — the SNAP pre-processing chain (orbit correction, noise removal, calibration, terrain flattening, terrain correction) must run first
- You need spatial speckle suppression for a single-scene analysis without a multi-temporal stack — the temporal median compositing approach requires multiple acquisitions and offers no benefit for single-date data

## Inputs

- Temporal stack of VH gamma_0 backscatter GeoTIFF (e.g. ALB_gamma0_VH.tif)
- Temporal stack of VV gamma_0 backscatter GeoTIFF (e.g. ALB_gamma0_VV.tif)
- CSV of Sentinel-1 acquisition dates (e.g. ALB_acquisition_dates.csv)
- Shapefile of example plot centers (e.g. ALB_example_plots.shp)
- CSV of reference plot mean values (e.g. S1_mean.csv) for validation

## Outputs

- Per-pixel median backscatter rasters for yearly, summer, and winter subsets of VV and VH
- Cross-polarization metric rasters: VV-VH difference, VV/VH ratio, winter-minus-summer differences per polarization
- Aggregated summary-statistic raster file (pixel_summary_statistics.tif)

## How to apply

Load the temporal stacks of VH and VV gamma_0 data (ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) and the acquisition dates file (ALB_acquisition_dates.csv) into R using the raster package. Use the Sentinel-1 acquisition dates to assign each acquisition to yearly, summer, and winter subsets per pixel, then compute per-pixel median backscatter for each subset and polarization so that temporal aggregation implicitly smooths speckle without spatial filtering — no additional spatial speckle filter is applied, avoiding further loss of geometric detail. Derive cross-polarization metrics per pixel: the VV-VH difference, the VV/VH ratio, and the winter-minus-summer differences for each polarization. Write the resulting rasters to pixel_summary_statistics.tif using rgdal/raster drivers, and check they are grid-aligned with the input gamma_0 stacks. These structural heterogeneity indices feed the canonical radar axes used in biodiversity modelling.

## Related tools

- **R** (Environment in which the pixel- and neighborhood-based summary statistics are computed) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Loading and manipulating the VH/VV gamma_0 temporal stacks and deriving per-pixel statistics)
- **rgdal** (Writing the derived summary-statistic rasters to GeoTIFF output)
- **glcm** (Minimum required R package listed alongside the summary-statistics workflow)
- **rgeos** (Minimum required R package for handling plot-center shapefile geometry)

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r
```

## Evaluation signals

- Derived statistic rasters are grid-aligned with the input gamma_0 stacks (same extent, resolution, CRS)
- Plot-level mean values extracted from the derived rasters are consistent with the reference plot means in S1_mean.csv
- Each acquisition is correctly assigned to yearly, summer, and winter subsets based on ALB_acquisition_dates.csv
- No spatial speckle filter was applied — spatial resolution remains at the 10 m gamma_0 pixel spacing
- All per-pixel statistics (medians, differences, ratios) exist for both polarizations across all temporal subsets

## Limitations

- C-band SAR has weak permeability through overstorey layers, so the understorey is poorly characterized, limiting prediction of species richness and phylogenetic diversity for groups driven by understorey development
- Structural heterogeneity is captured via standard deviation of yearly and summer backscatter feeding a canonical radar axis; this axis-based summary may not resolve all structural facets
- Method was developed and validated for temperate forest ecosystems of Central Europe; generality in other forest types is untested

## Evidence

- [readme] Temporal aggregation smoothing: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in space, thus not causing any further loss of geometric detail"
- [readme] Required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Demo input files: "Two temporal stacks of VH and VV gamma_0 data: ALB_gamma0_VH.tif and ALB_gamma0_VV.tif"
- [methods] R-based workflow: "Derivation of pixel- and neighborhood-based summary statistics: This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was"
