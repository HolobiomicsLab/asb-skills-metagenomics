---
name: raster-stack-temporal-analysis-in-r
description: Use when you have temporal stacks of geocoded Sentinel-1 gamma_0 rasters
  (VH and VV) plus acquisition dates, and need per-pixel seasonal/temporal summary
  statistics for downstream ecological modelling.
when_to_use_negative:
- The input imagery is raw Sentinel-1 GRDH digital numbers not yet processed to gamma_0
  — first run the SNAP/GPT pre-processing chain (precise orbit, noise removal, calibration,
  terrain flattening, range-doppler correction).
- Spatial speckle filtering is explicitly required (e.g. for single-acquisition scenes)
  — this skill deliberately smooths speckle in time via multi-acquisition medians
  instead of in space.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: R
  role: Environment in which the pixel- and neighborhood-based summary statistics
    from temporal gamma_0 stacks are computed
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Loads and manipulates the VH/VV gamma_0 temporal stacks and writes the summary
    statistic rasters
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgdal
  role: Raster driver support for reading/writing GeoTIFF inputs and pixel_summary_statistics.tif
    output
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgeos
  role: Minimum required R package for the spatial/vector handling accompanying the
    statistics derivation
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: glcm
  role: Minimum required R package for neighborhood-based texture statistics alongside
    the pixel-based statistics
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:ec198cadaad7110cca0e65778be53b81aa9cd7ba0de63955eb39750c1c61a790
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
yes_rate: 1.0
n_claims: 10
---

# Raster stack temporal analysis in R

## Summary

Computes pixel-based summary statistics (medians, VV-VH differences, VV/VH ratios, seasonal differences) from temporal stacks of Sentinel-1 gamma_0 VH/VV backscatter rasters in R, using multi-acquisition medians to smooth speckle implicitly in time rather than by spatial filtering.

## When to use

Use when you have temporal stacks of geocoded Sentinel-1 gamma_0 rasters (VH and VV) plus acquisition dates, and need per-pixel seasonal/temporal summary statistics for downstream ecological modelling.

## When NOT to use

- The input imagery is raw Sentinel-1 GRDH digital numbers not yet processed to gamma_0 — first run the SNAP/GPT pre-processing chain (precise orbit, noise removal, calibration, terrain flattening, range-doppler correction).
- Spatial speckle filtering is explicitly required (e.g. for single-acquisition scenes) — this skill deliberately smooths speckle in time via multi-acquisition medians instead of in space.

## Inputs

- Temporal stack of VH gamma_0 rasters (e.g. ALB_gamma0_VH.tif)
- Temporal stack of VV gamma_0 rasters (e.g. ALB_gamma0_VV.tif)
- Sentinel-1 acquisition dates table (e.g. ALB_acquisition_dates.csv)
- Optional plot-centre shapefile (e.g. ALB_example_plots.*) and reference plot means (S1_mean.csv) for validation

## Outputs

- Per-pixel median backscatter rasters for yearly, summer, and winter subsets of VV and VH
- Cross-polarization statistic rasters: VV-VH difference, VV/VH ratio, and winter-minus-summer differences per polarization
- Summary statistic raster stack written to pixel_summary_statistics.tif

## How to apply

Load the VH and VV gamma_0 temporal stacks (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) and the acquisition-dates table (ALB_acquisition_dates.csv) into R with the raster package. Use the Sentinel-1 acquisition dates to assign each acquisition in the stack to yearly, summer, and winter subsets per pixel. Compute per-pixel median backscatter for the yearly, summer, and winter subsets of each polarization — temporal aggregation of medians smooths speckle without spatial filtering, preserving geometric detail. Derive cross-polarization metrics per pixel: the VV-VH difference, the VV/VH ratio, and winter-minus-summer differences per polarization. Write the summary statistic rasters (e.g. to pixel_summary_statistics.tif) with rgdal/raster drivers. This derivation of pixel-based statistics builds on the SNAP/GPT pre-processing chain that converted GRDH digital numbers to gamma_0 via precise orbit application, border-noise and thermal-noise removal, radiometric calibration, radiometric terrain flattening (SRTM 1 sec DEM), and range-doppler terrain correction.

## Related tools

- **R** (Environment in which the pixel- and neighborhood-based summary statistics from temporal gamma_0 stacks are computed) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Loads and manipulates the VH/VV gamma_0 temporal stacks and writes the summary statistic rasters) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgdal** (Raster driver support for reading/writing GeoTIFF inputs and pixel_summary_statistics.tif output) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgeos** (Minimum required R package for the spatial/vector handling accompanying the statistics derivation) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **glcm** (Minimum required R package for neighborhood-based texture statistics alongside the pixel-based statistics) — https://github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r  # source the script after pointing it at the directory with ALB_gamma0_VH.tif, ALB_gamma0_VV.tif and ALB_acquisition_dates.csv
```

## Evaluation signals

- Derived statistic rasters are grid-aligned (extent, resolution, CRS) with the input gamma_0 stacks
- Plot-level means extracted from the derived rasters are consistent with the reference plot means in S1_mean.csv
- Output pixel_summary_statistics.tif is written successfully with rgdal/raster drivers and readable back in R
- Seasonal assignment of acquisitions (yearly/summer/winter) matches the dates listed in ALB_acquisition_dates.csv
- No spatial speckle filter was applied — the median-based temporal aggregation preserves the original 10 m geometric detail

## Limitations

- C-band SAR has weak permeability through overstorey layers, so metrics are poor for understorey-driven properties (e.g. species richness/phylogenetic diversity of understorey-driven taxa)
- Speckle smoothing is only temporal: a dense multi-temporal stack is required; single-acquisition data would remain unsmoothed
- Radiometric terrain flattening and range-doppler correction depend on DEM quality (SRTM 1 sec DEM used here) and precise orbit availability (available ~two weeks after data take)

## Evidence

- [readme] Temporal aggregation smooths speckle without spatial filtering: "However, due to the approach taken in this manuscript, that is, calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time"
- [methods] Derivation of pixel- and neighborhood-based summary statistics: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [other] Standard deviation of yearly and summer backscatter as structural heterogeneity indices: [evidence span withheld — anchor status: not_found]
- [readme] gamma_0 output in geographic coordinates at 10 m: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
