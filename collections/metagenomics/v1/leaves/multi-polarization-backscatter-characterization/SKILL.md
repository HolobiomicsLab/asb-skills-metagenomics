---
name: multi-polarization-backscatter-characterization
description: Use when Sentinel-1 gamma_0 temporal stacks (e.g. VH/VV GeoTIFFs such
  as ALB_gamma0_VH.tif and ALB_gamma0_VV.tif) must be summarized into neighborhood
  texture variables representing habitat structural heterogeneity.
when_to_use_negative:
- Input is still raw GRDH digital numbers in slant-range coordinates — apply the SNAP
  gamma_0 processing chain (precise orbit, border/thermal noise removal, radiometric
  calibration, terrain flattening, range-doppler terrain correction) first
- The goal is pixel-based mean backscatter statistics per plot (e.g. the S1_mean.csv
  temporal medians) rather than neighborhood texture heterogeneity layers
- The target variable depends on understorey structure that C-band backscatter cannot
  characterize due to weak permeability through overstorey layers
tools:
- name: R
  role: Environment in which the pixel- and neighborhood-based summary statistics
    over the gamma_0 stacks are performed
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: glcm
  role: Computes grey-level co-occurrence matrix textures (contrast, dissimilarity,
    homogeneity, orderliness indices) with 32 grey levels over a 9x9-pixel window
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Loads the VH/VV gamma_0 temporal stacks and writes the GLCM index layers as
    GeoTIFF rasters preserving the 10 m grid
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgdal
  role: GeoTIFF I/O backend required alongside raster for reading/writing the gamma_0
    and texture layers
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgeos
  role: Required minimum R package installed in addition for the spatial summary statistics
    workflow
  repo: https://github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:e9e6d5745d496d12e56adf5531110e50e0cfecb00484395039357312501f9e19
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multi-polarization-backscatter-characterization

## Summary

Transforms temporal stacks of Sentinel-1 VH and VV gamma_0 backscatter into GLCM texture layers (contrast-group and orderliness-group indices) that quantify horizontal structural heterogeneity of forest habitats, using R's glcm and raster packages.

## When to use

Use when Sentinel-1 gamma_0 temporal stacks (e.g. VH/VV GeoTIFFs such as ALB_gamma0_VH.tif and ALB_gamma0_VV.tif) must be summarized into neighborhood texture variables representing habitat structural heterogeneity.

## When NOT to use

- Input is still raw GRDH digital numbers in slant-range coordinates — apply the SNAP gamma_0 processing chain (precise orbit, border/thermal noise removal, radiometric calibration, terrain flattening, range-doppler terrain correction) first
- The goal is pixel-based mean backscatter statistics per plot (e.g. the S1_mean.csv temporal medians) rather than neighborhood texture heterogeneity layers
- The target variable depends on understorey structure that C-band backscatter cannot characterize due to weak permeability through overstorey layers

## Inputs

- Temporal stack of VH gamma_0 rasters (GeoTIFF, e.g. ALB_gamma0_VH.tif)
- Temporal stack of VV gamma_0 rasters (GeoTIFF, e.g. ALB_gamma0_VV.tif)
- Terrain-corrected gamma_0 raster grid at 10 m pixel spacing

## Outputs

- GLCM contrast-group texture layers (contrast, dissimilarity, homogeneity) per polarization
- GLCM orderliness-group texture layers per polarization
- GeoTIFF texture index rasters on the 10 m pixel grid matching the input gamma_0 stacks

## How to apply

Load the temporal stacks of VH and VV gamma_0 rasters (e.g. ALB_gamma0_VH.tif, ALB_gamma0_VV.tif) in R with the raster package. Run the glcm package on each stack, computing grey-level co-occurrence textures with 32 grey levels and a 9x9-pixel moving window. Extract the contrast-group indices (contrast, dissimilarity, homogeneity) and the orderliness-group indices from the co-occurrence statistics. Write the resulting GLCM index layers for both polarizations to output GeoTIFF rasters, preserving the 10 m pixel grid of the terrain-corrected gamma_0 inputs. The rationale is that contrast and orderliness capture spatial (horizontal) heterogeneity that correlates with gradients of forest structure, e.g. the third radar canonical axis.

## Related tools

- **R** (Environment in which the pixel- and neighborhood-based summary statistics over the gamma_0 stacks are performed) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **glcm** (Computes grey-level co-occurrence matrix textures (contrast, dissimilarity, homogeneity, orderliness indices) with 32 grey levels over a 9x9-pixel window) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Loads the VH/VV gamma_0 temporal stacks and writes the GLCM index layers as GeoTIFF rasters preserving the 10 m grid) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgdal** (GeoTIFF I/O backend required alongside raster for reading/writing the gamma_0 and texture layers) — https://github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgeos** (Required minimum R package installed in addition for the spatial summary statistics workflow) — https://github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
texture <- glcm::glcm(raster::stack("ALB_gamma0_VH.tif"), window = c(9,9), n_grey = 32, statistics = c("contrast", "dissimilarity", "homogeneity", "entropy"))
```

## Evaluation signals

- Every expected GLCM index layer exists for both VH and VV polarizations
- Each output texture raster matches the extent and 10 m resolution of the corresponding input gamma_0 stack
- GLCM computation used 32 grey levels and a 9x9-pixel moving window
- Contrast and orderliness texture measures show strong correlations with the third radar canonical axis representing the gradient of structural heterogeneity
- Texture rasters preserve the geographic coordinates and pixel spacing established by range-doppler terrain correction

## Limitations

- C-band SAR has weak permeability through overstorey layers due to short wavelengths, limiting characterization of the understorey and prediction of richness/phylogenetic diversity for groups driven by understorey development
- Speckle is smoothed implicitly in time (median backscatter over multiple acquisitions) rather than by spatial filtering, to avoid loss of geometric detail
- Texture layers inherit terrain effects unless the inputs are radiometrically terrain-flattened gamma_0 on a DEM such as SRTM 1 sec

## Evidence

- [methods] Derivation of pixel- and neighborhood-based summary statistics: "Derivation of pixel- and neighborhood-based summary statistics"
- [readme] Required R packages for GLCM workflow: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Temporal gamma_0 stacks processed in R: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] 10 m geographic grid of gamma_0 output: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [readme] Input file naming for VH/VV stacks: "Two temporal stacks of VH and VV gamma_0 data: ALB_gamma0_VH.tif and ALB_gamma0_VV.tif"
