---
name: r-raster-stack-processing
description: Use when raster time-series stacks (e.g. multi-date Sentinel-1 gamma_0
  GeoTIFFs) must be read in R and summarised into per-pixel statistics or neighborhood
  texture metrics (GLCM contrast/dissimilarity, orderliness/entropy) for habitat-structure
  mapping.
when_to_use_negative:
- Input is a raw uncalibrated GRDH product in slant-range DN — it must first be processed
  to terrain-corrected gamma_0 via SNAP/GPT (apply orbit, remove border/thermal noise,
  calibrate, terrain flatten, terrain correct)
- Spatial speckle filtering is desired as an explicit step — this workflow intentionally
  smooths speckle implicitly in time via multi-acquisition median backscatter instead
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: R
  role: Environment in which raster stacks are loaded and summary statistics derived
- name: raster
  role: Reads temporal gamma_0 GeoTIFF stacks and preserves extent/resolution in outputs
- name: glcm
  role: Computes grey-level co-occurrence texture indices with 32 grey levels and
    9x9-pixel moving window
- name: rgdal
  role: Minimum required package for GeoTIFF I/O in the workflow
- name: rgeos
  role: Minimum required spatial package installed alongside raster/rgdal/glcm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:5fa8a55ca986afbb9a987222b3487b72b82492e60fce9d847ace73e342c1456d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glcm
  - raster
  - rgdal
  - rgeos
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# r-raster-stack-processing

## Summary

Load and process temporal raster stacks (e.g. Sentinel-1 VH and VV gamma_0 GeoTIFFs) in R using the raster package to derive pixel- and neighborhood-based summary statistics such as GLCM texture indices on the original 10 m grid.

## When to use

Use when raster time-series stacks (e.g. multi-date Sentinel-1 gamma_0 GeoTIFFs) must be read in R and summarised into per-pixel statistics or neighborhood texture metrics (GLCM contrast/dissimilarity, orderliness/entropy) for habitat-structure mapping.

## When NOT to use

- Input is a raw uncalibrated GRDH product in slant-range DN — it must first be processed to terrain-corrected gamma_0 via SNAP/GPT (apply orbit, remove border/thermal noise, calibrate, terrain flatten, terrain correct)
- Spatial speckle filtering is desired as an explicit step — this workflow intentionally smooths speckle implicitly in time via multi-acquisition median backscatter instead

## Inputs

- Temporal stacks of VH and VV gamma_0 GeoTIFF rasters (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) at 10 m pixel spacing
- Sentinel-1 acquisition dates CSV (e.g. ALB_acquisition_dates.csv)
- Shapefile of plot centres (e.g. ALB_example_plots.*)

## Outputs

- GLCM texture index GeoTIFF layers (contrast, dissimilarity, homogeneity; orderliness/entropy) for VH and VV stacks on the original 10 m grid
- Per-pixel and neighborhood summary statistic rasters derived from the gamma_0 temporal stacks

## How to apply

Load the temporal stacks of VH and VV gamma_0 rasters (e.g. ALB_gamma0_VH.tif and ALB_gamma0_VV.tif) with the R raster package, which requires rgeos, raster, rgdal and glcm as minimum packages. For neighborhood texture, run the glcm package on each gamma_0 stack using 32 grey levels and a 9x9-pixel moving window, then extract contrast-group indices (contrast, dissimilarity, homogeneity) and orderliness-group indices. Speckle smoothing is deliberately done in time rather than space — median backscatter over multiple acquisitions — so no spatial speckle filter is applied and the 10 m pixel grid of the terrain-corrected gamma_0 inputs is preserved. Write the resulting statistic layers back to GeoTIFF, and validate that each output raster matches the extent and resolution of the corresponding input gamma_0 stack.

## Related tools

- **R** (Environment in which raster stacks are loaded and summary statistics derived)
- **raster** (Reads temporal gamma_0 GeoTIFF stacks and preserves extent/resolution in outputs)
- **glcm** (Computes grey-level co-occurrence texture indices with 32 grey levels and 9x9-pixel moving window)
- **rgdal** (Minimum required package for GeoTIFF I/O in the workflow)
- **rgeos** (Minimum required spatial package installed alongside raster/rgdal/glcm)

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r
```

## Evaluation signals

- Every expected GLCM index layer (contrast-group and orderliness-group) exists for both VH and VV polarizations
- Each output raster matches the extent and resolution of the corresponding input gamma_0 stack (10 m pixel grid preserved)
- GLCM computation used 32 grey levels and a 9x9-pixel moving window as specified
- Output layers are written as GeoTIFF rasters readable back by raster::stack()

## Limitations

- C-band SAR weakly permeates the overstorey, so understorey-driven variables (e.g. species richness/phylogenetic diversity of understorey-driven groups) are poorly represented in radar-derived layers
- Speckle smoothing is implicit (temporal median over multiple acquisitions), so single-acquisition stacks retain speckle and lose the implicit denoising benefit

## Evidence

- [methods] This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] smoothing of speckle effects takes place implicitly in the time, instead of in the space, thus not causing any further loss of geometric detail: [evidence span withheld — anchor status: not_found]
- [readme] The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing.: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
