---
name: sar-gamma0-raster-analysis
description: Use when you have terrain-corrected Sentinel-1 gamma_0 backscatter stacks
  (e.g. GeoTIFFs such as ALB_gamma0_VH.tif / ALB_gamma0_VV.tif) and need raster variables
  describing horizontal (spatial) forest structure rather than per-pixel backscatter.
when_to_use_negative:
- Input is still raw Sentinel-1 GRDH digital numbers rather than calibrated, terrain-flattened
  and orthorectified gamma_0 — pre-process via SNAP/GPT first (precise orbit, border/thermal
  noise removal, radiometric calibration, terrain flattening, range-doppler correction)
- You only need per-pixel backscatter or per-plot mean values rather than neighborhood-based
  spatial heterogeneity metrics
- 'Your question concerns understorey structure: C-band SAR has weak permeability
  through the overstorey, so texture from Sentinel-1 poorly predicts understorey-driven
  richness/phylogenetic diversity'
edam_operation: http://edamontology.org/operation_3443
tools:
- name: R
  role: Environment in which gamma_0 temporal stacks are loaded and the texture statistics
    are computed
- name: glcm
  role: R package that computes grey-level co-occurrence texture indices over a moving
    window (32 grey levels, 9x9 pixels)
- name: raster
  role: R package used to load, manipulate and write the gamma_0 and GLCM GeoTIFF
    rasters
- name: rgdal
  role: Minimum required R package for raster I/O (reading/writing GeoTIFFs)
- name: rgeos
  role: Minimum required R package listed for the pixel- and neighborhood-based summary
    statistics workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:16448e991924628280dc2feab4cbe578658e30882f00771b50529d6500ed349b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glcm
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sar-gamma0-raster-analysis

## Summary

Computes grey-level co-occurrence matrix (GLCM) texture indices (contrast/dissimilarity/homogeneity and orderliness/entropy) over moving windows on temporal stacks of terrain-corrected Sentinel-1 gamma_0 VH and VV rasters, to quantify horizontal structural heterogeneity of forest habitats.

## When to use

Use when you have terrain-corrected Sentinel-1 gamma_0 backscatter stacks (e.g. GeoTIFFs such as ALB_gamma0_VH.tif / ALB_gamma0_VV.tif) and need raster variables describing horizontal (spatial) forest structure rather than per-pixel backscatter.

## When NOT to use

- Input is still raw Sentinel-1 GRDH digital numbers rather than calibrated, terrain-flattened and orthorectified gamma_0 — pre-process via SNAP/GPT first (precise orbit, border/thermal noise removal, radiometric calibration, terrain flattening, range-doppler correction)
- You only need per-pixel backscatter or per-plot mean values rather than neighborhood-based spatial heterogeneity metrics
- Your question concerns understorey structure: C-band SAR has weak permeability through the overstorey, so texture from Sentinel-1 poorly predicts understorey-driven richness/phylogenetic diversity

## Inputs

- Temporal stack of VH gamma_0 rasters (GeoTIFF, e.g. ALB_gamma0_VH.tif) at 10 m pixel spacing
- Temporal stack of VV gamma_0 rasters (GeoTIFF, e.g. ALB_gamma0_VV.tif) at 10 m pixel spacing
- Sentinel-1 acquisition dates table (CSV, e.g. ALB_acquisition_dates.csv)

## Outputs

- GeoTIFF raster layers of contrast-group GLCM indices (contrast, dissimilarity, homogeneity) for VH and VV
- GeoTIFF raster layers of orderliness-group GLCM indices (e.g. entropy) for VH and VV
- Output rasters matching the extent and 10 m resolution of the corresponding input gamma_0 stack

## How to apply

Load the temporal stacks of VH and VV gamma_0 rasters (e.g. ALB_gamma0_VH.tif and ALB_gamma0_VV.tif) in R with the raster package, preserving the 10 m pixel grid of the terrain-corrected inputs. Run the glcm package on each stack with 32 grey levels and a 9x9-pixel moving window. Extract the contrast-group indices (contrast, dissimilarity, homogeneity) and the orderliness-group indices (e.g. entropy) from the co-occurrence statistics. Write the resulting texture index layers for both polarizations to output GeoTIFFs on the same extent/resolution as the input gamma_0 stacks. Rationale: texture measures capture spatial heterogeneity that correlates with structural gradients (the third radar canonical axis in the study), complementing per-pixel median backscatter computed over multiple acquisitions, where speckle is smoothed in time rather than in space.

## Related tools

- **R** (Environment in which gamma_0 temporal stacks are loaded and the texture statistics are computed)
- **glcm** (R package that computes grey-level co-occurrence texture indices over a moving window (32 grey levels, 9x9 pixels))
- **raster** (R package used to load, manipulate and write the gamma_0 and GLCM GeoTIFF rasters)
- **rgdal** (Minimum required R package for raster I/O (reading/writing GeoTIFFs))
- **rgeos** (Minimum required R package listed for the pixel- and neighborhood-based summary statistics workflow)

## Examples

```
Rscript Sentinel1_pixel_neighborhood_statistics.r  # with glcm(stack, n_grey=32, window=c(9,9), statistics=c('contrast','dissimilarity','homogeneity','entropy')) on ALB_gamma0_VH.tif / ALB_gamma0_VV.tif
```

## Evaluation signals

- Every expected GLCM index layer exists for both VH and VV polarizations (contrast, dissimilarity, homogeneity from the contrast group; orderliness/entropy indices from the orderliness group)
- Each output GLCM raster matches the extent and 10 m pixel resolution of the corresponding input gamma_0 stack
- Texture values are finite (no NA propagation across the full raster) and plausible for the chosen 32-grey-level quantization over a 9x9 window
- Texture layers show meaningful correlation with expected structural gradients (in the study, GLCM contrast/orderliness strongly correlated with the third radar canonical axis of structural heterogeneity)
- No explicit spatial speckle filter was applied to the inputs — speckle smoothing is implicit in the temporal median over multiple acquisitions

## Limitations

- C-band Sentinel-1 has weak permeability through overstorey layers due to short wavelengths, so understorey-driven biodiversity components (species richness, phylogenetic diversity of understorey-associated groups) are poorly predicted
- Explicit spatial speckle filtering is intentionally avoided; smoothing relies on temporal medians over multiple acquisitions, so single-acquisition inputs retain speckle
- Neighborhood statistics assume a consistent 10 m grid from the SNAP range-doppler terrain correction with SRTM 1 sec DEM

## Evidence

- [methods] GLCM texture over temporal gamma_0 stacks: [evidence span withheld — anchor status: not_found]
- [methods] texture indices quantify horizontal heterogeneity: [evidence span withheld — anchor status: not_found]
- [readme] inputs are gamma_0 temporal stacks in R: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] output grid from terrain correction: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
