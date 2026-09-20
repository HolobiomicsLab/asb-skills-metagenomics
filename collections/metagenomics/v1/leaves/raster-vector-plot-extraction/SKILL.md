---
name: raster-vector-plot-extraction
description: Use when Sentinel-1 gamma_0 summary rasters must be sampled at field-plot
  center locations (e.g. ALB_example_plots.* shapefile) to derive plot-level means
  or validate them against reference values such as S1_mean.csv.
when_to_use_negative:
- Inputs are raw Sentinel-1 GRDH digital numbers in slant-range coordinates; these
  must first be processed to gamma_0 via the SNAP/GPT chain (precise orbit, border/thermal
  noise removal, calibration, terrain flattening, range-doppler correction).
- The goal is biodiversity modeling itself (e.g. R² or NMDS of assemblage composition)
  rather than extracting plot-level raster values from the summary statistics.
- Input is already a table of plot-level means (e.g. S1_mean.csv); no raster sampling
  is needed.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: R
  role: Runtime environment in which raster sampling and plot-mean validation of the
    gamma_0 summary statistics are performed
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Reading the gamma_0 and summary statistic rasters and extracting values at
    plot-center locations
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgdal
  role: Reading/writing the ALB_example_plots.* plot-center shapefile and geotiff
    rasters for extraction
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgeos
  role: Vector geometry handling of plot-center shapefiles during extraction
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:1ed92a4b87923f829ed6b932fb3beca4ddfbafc8a3c759b89d724f0516cded94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# raster-vector-plot-extraction

## Summary

Samples Sentinel-1 gamma_0 pixel-based summary statistic rasters (medians, VV-VH differences, VV/VH ratios) at forest plot-center vector locations in R, aggregating 10 m pixels to 1-ha plot means and validating them against reference values.

## When to use

Use when Sentinel-1 gamma_0 summary rasters must be sampled at field-plot center locations (e.g. ALB_example_plots.* shapefile) to derive plot-level means or validate them against reference values such as S1_mean.csv.

## When NOT to use

- Inputs are raw Sentinel-1 GRDH digital numbers in slant-range coordinates; these must first be processed to gamma_0 via the SNAP/GPT chain (precise orbit, border/thermal noise removal, calibration, terrain flattening, range-doppler correction).
- The goal is biodiversity modeling itself (e.g. R² or NMDS of assemblage composition) rather than extracting plot-level raster values from the summary statistics.
- Input is already a table of plot-level means (e.g. S1_mean.csv); no raster sampling is needed.

## Inputs

- Temporal stacks of VH and VV gamma_0 data: ALB_gamma0_VH.tif and ALB_gamma0_VV.tif (10 m, geographic coordinates)
- Derived pixel-based summary statistic rasters, e.g. pixel_summary_statistics.tif
- Shapefile of centers of example plots: ALB_example_plots.*
- Sentinel-1 acquisition dates: ALB_acquisition_dates.csv
- Reference calculated mean values of 1-ha plots: S1_mean.csv

## Outputs

- Plot-level mean backscatter statistic values per 1-ha plot (per polarization and per statistic)
- Validation comparison of extracted plot means against S1_mean.csv reference means

## How to apply

Load the derived pixel-based summary statistic rasters (e.g. pixel_summary_statistics.tif, built from the ALB_gamma0_VH.tif and ALB_gamma0_VV.tif 10 m gamma_0 stacks) and the plot-center shapefile ALB_example_plots.* in R using the raster and rgdal packages. Extract per-pixel statistic values at each plot center (1-ha plots) and average them into plot-level means. Compare the extracted plot means against the reference means in S1_mean.csv to validate the extraction and the upstream statistic derivation. Confirm the raster grid alignment with the input gamma_0 stacks so that extraction hits the correct pixels.

## Related tools

- **R** (Runtime environment in which raster sampling and plot-mean validation of the gamma_0 summary statistics are performed) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Reading the gamma_0 and summary statistic rasters and extracting values at plot-center locations) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgdal** (Reading/writing the ALB_example_plots.* plot-center shapefile and geotiff rasters for extraction) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgeos** (Vector geometry handling of plot-center shapefiles during extraction) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r
```

## Evaluation signals

- Extracted plot-level means are consistent with the reference plot means in S1_mean.csv.
- Derived statistic rasters are grid-aligned with the input gamma_0 stacks (same extent, resolution, CRS).
- Every plot feature in ALB_example_plots.* yields a non-NA value from the 10 m gamma_0 grid.
- Extracted gamma_0 values fall in physically plausible backscatter ranges (post terrain flattening/calibration, not raw DN).
- Plot-mean aggregation covers all expected statistics (yearly/summer/winter medians, VV-VH difference, VV/VH ratio, winter-minus-summer differences).

## Limitations

- C-band SAR has weak permeability through overstorey layers, so extracted plot means carry little understorey information; understorey-driven signals are poorly represented.
- Speckle is smoothed only implicitly in time (via per-pixel medians over acquisitions), not spatially, so single-acquisition or single-pixel extractions remain noisy.
- Extraction assumes geocoded gamma_0 in geographic coordinates at 10 m pixel spacing; misalignment with uncorrected slant-range products would invalidate plot-value sampling.

## Evidence

- [methods] This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] 4) Calculated mean values of all our 1-ha plots: S1_mean.csv: "4) Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [readme] The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing.: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
