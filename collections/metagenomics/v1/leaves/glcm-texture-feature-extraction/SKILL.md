---
name: glcm-texture-feature-extraction
description: Use when Sentinel-1 gamma_0 backscatter stacks (e.g. ALB_gamma0_VH.tif
  / ALB_gamma0_VV.tif) must be converted into spatial-heterogeneity texture predictor
  variables for habitat-structure or biodiversity modelling on a 10 m grid.
when_to_use_negative:
- Input is a single-acquisition SAR image with heavy speckle where a dedicated spatial
  speckle filter is intended — this workflow relies on temporal stacks, since median
  backscatter over multiple acquisitions smooths speckle implicitly in time instead
  of in space
- The analysis targets processes driven by understorey development (e.g. species richness
  or phylogenetic diversity of groups sensitive to understorey), which C-band texture
  of gamma_0 does not capture well
- A pre-computed feature/texture table already exists and only downstream modelling
  is needed
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: glcm
  role: Computes grey-level co-occurrence texture statistics (contrast/dissimilarity/homogeneity,
    entropy) with 32 grey levels over a 9x9-pixel window on the gamma_0 stacks
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Loads the VH/VV gamma_0 temporal stacks in R and handles writing the GLCM
    index layers to GeoTIFF output rasters
- name: rgdal
  role: Minimum required R package supporting raster I/O of the GeoTIFF gamma_0 inputs
    and outputs
- name: rgeos
  role: Minimum required R package listed alongside raster/rgdal/glcm for the pixel-
    and neighborhood-based statistics workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:048c0415cf07bd99866645b17ac8ff2166dc2ae831c016ecaba940290b5b6e39
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
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# glcm-texture-feature-extraction

## Summary

Computes grey-level co-occurrence matrix (GLCM) texture metrics — contrast-group indices (contrast, dissimilarity, homogeneity) and orderliness-group indices (entropy) — over 9x9-pixel moving windows with 32 grey levels on temporal stacks of Sentinel-1 gamma_0 VH and VV rasters, quantifying horizontal structural heterogeneity of forest habitats.

## When to use

Use when Sentinel-1 gamma_0 backscatter stacks (e.g. ALB_gamma0_VH.tif / ALB_gamma0_VV.tif) must be converted into spatial-heterogeneity texture predictor variables for habitat-structure or biodiversity modelling on a 10 m grid.

## When NOT to use

- Input is a single-acquisition SAR image with heavy speckle where a dedicated spatial speckle filter is intended — this workflow relies on temporal stacks, since median backscatter over multiple acquisitions smooths speckle implicitly in time instead of in space
- The analysis targets processes driven by understorey development (e.g. species richness or phylogenetic diversity of groups sensitive to understorey), which C-band texture of gamma_0 does not capture well
- A pre-computed feature/texture table already exists and only downstream modelling is needed

## Inputs

- Temporal stack of VH gamma_0 rasters (e.g. ALB_gamma0_VH.tif), 10 m pixel grid
- Temporal stack of VV gamma_0 rasters (e.g. ALB_gamma0_VV.tif), 10 m pixel grid
- R environment with raster, rgdal, rgeos and glcm packages installed

## Outputs

- GeoTIFF raster per GLCM index and polarization (contrast-group: contrast, dissimilarity, homogeneity; orderliness-group: entropy)
- Texture index layers matching the extent and 10 m resolution of the input gamma_0 stacks

## How to apply

Start from temporal stacks of terrain-corrected Sentinel-1 gamma_0 rasters for VH and VV polarization (e.g. ALB_gamma0_VH.tif and ALB_gamma0_VV.tif) loaded in R with the raster package. Run the glcm package on each gamma_0 stack using 32 grey levels and a 9x9-pixel moving window, so that speckle is smoothed spatially at neighbourhood scale rather than by a speckle filter. Extract the contrast-group GLCM indices (contrast, dissimilarity, homogeneity) and the orderliness-group indices (entropy) from the co-occurrence statistics, separately for VH and VV. Write each texture index layer to an output GeoTIFF preserving the 10 m pixel grid of the terrain-corrected gamma_0 inputs. The rationale is that these texture measures quantify spatial (horizontal) heterogeneity and showed strong correlations with the third radar canonical axis representing a gradient of structural heterogeneity.

## Related tools

- **glcm** (Computes grey-level co-occurrence texture statistics (contrast/dissimilarity/homogeneity, entropy) with 32 grey levels over a 9x9-pixel window on the gamma_0 stacks) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Loads the VH/VV gamma_0 temporal stacks in R and handles writing the GLCM index layers to GeoTIFF output rasters)
- **rgdal** (Minimum required R package supporting raster I/O of the GeoTIFF gamma_0 inputs and outputs)
- **rgeos** (Minimum required R package listed alongside raster/rgdal/glcm for the pixel- and neighborhood-based statistics workflow)

## Examples

```
library(glcm); library(raster); tex <- glcm(raster("ALB_gamma0_VH.tif"), window = c(9,9), n_grey = 32, statistics = c("contrast","dissimilarity","homogeneity","entropy"))
```

## Evaluation signals

- Every expected GLCM index layer exists for both polarizations (VH and VV), covering contrast-group (contrast, dissimilarity, homogeneity) and orderliness-group indices
- Each output raster matches the extent and resolution of the corresponding input gamma_0 stack, preserving the 10 m pixel grid
- GLCM computation used the intended parameters: 32 grey levels and a 9x9-pixel moving window
- Texture measures show strong correlations with the third radar canonical axis representing structural heterogeneity
- No separate spatial speckle filter was applied — speckle smoothing occurs implicitly via temporal median over multiple acquisitions

## Limitations

- Spatial speckle filtering is deliberately skipped: smoothing relies on median backscatter over multiple acquisitions in time, so single-date inputs would carry unsmoothed speckle and lose geometric detail if filtered
- C-band SAR has weak permeability through overstorey layers, so texture metrics reflect structure but cannot characterise the understorey; predictions of species richness and phylogenetic diversity for understorey-driven groups (e.g. some carabids, spiders, bats) were poor
- GLCM texture was validated against temperate forest ecosystems of Central Europe; generality in other forest types is untested

## Evidence

- [methods] Derivation of pixel- and neighborhood-based summary statistics: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [readme] Example input files: "1) Two temporal stacks of VH and VV gamma_0 data: ALB_gamma0_VH.tif and ALB_gamma0_VV.tif"
- [readme] Speckle handling rationale: "due to the approach taken in this manuscript, that is, calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in the"
