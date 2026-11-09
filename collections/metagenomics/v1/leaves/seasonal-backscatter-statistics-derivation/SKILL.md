---
name: seasonal-backscatter-statistics-derivation
description: Use when temporal stacks of Sentinel-1 gamma_0 (VH and VV) plus an acquisition-dates
  table are available and you need pixel-based seasonal backscatter summaries (medians,
  differences, ratios) as forest-structure predictors.
when_to_use_negative:
- Input is raw Sentinel-1 GRDH digital numbers rather than pre-processed, terrain-flattened
  gamma_0 stacks (run the SNAP/GPT preprocessing chain first)
- You need to characterize the understorey layer — C-band SAR has weak permeability
  through overstorey layers and performs poorly for groups driven by understorey development
- A spatially explicit single-date backscatter product is required — this skill produces
  temporally aggregated summary statistics, not instantaneous scenes
edam_operation: http://edamontology.org/operation_2939
edam_topics: []
tools:
- name: R
  role: Environment in which the pixel-based summary statistics script (Sentinel1_pixel_neighborhood_statistics.r)
    is sourced and executed
  repo: https://cran.r-project.org/
- name: raster
  role: Loading the VH/VV gamma_0 temporal stacks and handling per-pixel temporal
    subsetting and statistics
- name: rgdal
  role: Writing the derived summary-statistic rasters (pixel_summary_statistics.tif)
    via raster drivers
- name: rgeos
  role: Minimum required R package for vector handling when extracting/validating
    plot-level statistics
- name: glcm
  role: Minimum required R package supporting neighborhood-based texture statistics
    alongside the pixel-based statistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:0127983e16ab36d9c87af52c3e7a6e6706ae404ba0c12e0174c3c44188315894
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glcm
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# seasonal-backscatter-statistics-derivation

## Summary

Derives per-pixel seasonal (yearly, summer, winter) summary statistics — median VV/VH gamma_0, VV−VH difference, VV/VH ratio, and winter-minus-summer differences — from temporal stacks of Sentinel-1 gamma_0 in R, using temporal medians to smooth speckle instead of spatial filtering.

## When to use

Use when temporal stacks of Sentinel-1 gamma_0 (VH and VV) plus an acquisition-dates table are available and you need pixel-based seasonal backscatter summaries (medians, differences, ratios) as forest-structure predictors.

## When NOT to use

- Input is raw Sentinel-1 GRDH digital numbers rather than pre-processed, terrain-flattened gamma_0 stacks (run the SNAP/GPT preprocessing chain first)
- You need to characterize the understorey layer — C-band SAR has weak permeability through overstorey layers and performs poorly for groups driven by understorey development
- A spatially explicit single-date backscatter product is required — this skill produces temporally aggregated summary statistics, not instantaneous scenes

## Inputs

- Temporal stack of VH gamma_0 rasters (GeoTIFF, e.g. ALB_gamma0_VH.tif)
- Temporal stack of VV gamma_0 rasters (GeoTIFF, e.g. ALB_gamma0_VV.tif)
- Sentinel-1 acquisition dates table (CSV, e.g. ALB_acquisition_dates.csv)
- Reference plot means for validation (CSV, e.g. S1_mean.csv)
- Plot center shapefile (e.g. ALB_example_plots.*) for extraction/validation

## Outputs

- Per-pixel seasonal median gamma_0 rasters (yearly, summer, winter; per polarization VV and VH)
- Per-pixel VV−VH difference raster
- Per-pixel VV/VH ratio raster
- Per-pixel winter-minus-summer difference rasters per polarization
- pixel_summary_statistics.tif (raster stack of the derived statistics)

## How to apply

Load the VH and VV gamma_0 temporal stacks (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) and the acquisition dates (ALB_acquisition_dates.csv) into R with the raster package. Use the Sentinel-1 acquisition dates to assign each acquisition per pixel to yearly, summer, and winter subsets, then compute per-pixel median gamma_0 for each subset and polarization; temporal aggregation over multiple acquisitions smooths speckle implicitly, so no spatial speckle filter is applied (avoiding loss of geometric detail). Derive cross-polarization metrics per pixel: the VV−VH difference, the VV/VH ratio, and winter-minus-summer differences per polarization. Write the statistic rasters to pixel_summary_statistics.tif via rgdal/raster drivers, keeping the input grid alignment so plot-level extraction is possible.

## Related tools

- **R** (Environment in which the pixel-based summary statistics script (Sentinel1_pixel_neighborhood_statistics.r) is sourced and executed) — https://cran.r-project.org/
- **raster** (Loading the VH/VV gamma_0 temporal stacks and handling per-pixel temporal subsetting and statistics)
- **rgdal** (Writing the derived summary-statistic rasters (pixel_summary_statistics.tif) via raster drivers)
- **rgeos** (Minimum required R package for vector handling when extracting/validating plot-level statistics)
- **glcm** (Minimum required R package supporting neighborhood-based texture statistics alongside the pixel-based statistics)

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r  # with ALB_gamma0_VH.tif, ALB_gamma0_VV.tif, ALB_acquisition_dates.csv in the defined input directory
```

## Evaluation signals

- Derived statistic rasters are grid-aligned with the input gamma_0 stacks (same extent, resolution, CRS)
- Plot-level means extracted from the statistic rasters are consistent with reference plot means in S1_mean.csv
- All output GeoTIFFs written successfully to pixel_summary_statistics.tif with expected layer names (medians per season/polarization, VV−VH difference, VV/VH ratio, winter−summer differences)
- No spatial speckle filter applied — geometric detail retained at the 10 m gamma_0 pixel spacing
- Seasonal subsets (yearly/summer/winter) match assignments implied by ALB_acquisition_dates.csv

## Limitations

- Speckle smoothing occurs only in the time domain via medians over multiple acquisitions; if few acquisitions exist per season, medians remain noisy
- C-band SAR has weak permeability through overstorey layers, so these statistics poorly represent understorey structure (limiting species richness/phylogenetic diversity prediction for understorey-driven groups)
- Statistics were developed for temperate forest ecosystems of Central Europe; generality in other forest types is untested

## Evidence

- [methods] Compute per-pixel median backscatter for the yearly, summer, and winter subsets of each polarization (VV, VH), so that temporal aggregation implicitly smooths speckle without spatial filtering.: [evidence span withheld — anchor status: not_found]
- [methods] Derive cross-polarization metrics per pixel: the VV-VH difference and the VV/VH ratio, and the winter-minus-summer differences for each polarization.: [evidence span withheld — anchor status: not_found]
- [readme] This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [discussion] The shortcoming of Sentinel-1 data that we uncovered was in the prediction of species richness and phylogenetic diversity for groups that were more strongly driven by the development of the: "The shortcoming of Sentinel-1 data that we uncovered was in the prediction of species richness and phylogenetic diversity for groups that were more strongly driven by the development of the"
