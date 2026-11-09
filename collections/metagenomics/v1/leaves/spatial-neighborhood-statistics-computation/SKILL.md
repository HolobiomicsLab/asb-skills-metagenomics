---
name: spatial-neighborhood-statistics-computation
description: Use when you have terrain-corrected Sentinel-1 gamma_0 raster stacks
  (10 m pixels) and need spatial texture variables representing habitat structural
  heterogeneity for biodiversity models.
when_to_use_negative:
- Input rasters are raw GRDH digital numbers in slant-range coordinates rather than
  terrain-corrected gamma_0 — run the SNAP/GPT pre-processing chain first
- The analysis goal is vertical canopy structure (e.g. understorey or canopy height),
  where C-band Sentinel-1 has weak permeability and ALS is more appropriate
- A spatial speckle filter is explicitly required before texture analysis — this workflow
  relies on temporal (multi-acquisition median) speckle smoothing instead, to avoid
  loss of geometric detail
edam_operation: http://edamontology.org/operation_3443
tools:
- name: R
  role: Environment in which the pixel- and neighborhood-based summary statistics
    are computed
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: glcm
  role: Computes grey-level co-occurrence texture measures (32 grey levels, 9x9-pixel
    window) on the gamma_0 stacks
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: raster
  role: Loads the VH/VV gamma_0 temporal stacks and writes the GLCM texture layers
    to GeoTIFF on the original 10 m grid
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgdal
  role: Supports reading/writing georeferenced raster and shapefile data within the
    R workflow
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
- name: rgeos
  role: Supports the required minimum R package set for the spatial statistics workflow
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:35ce154ced52cc1b5f801c98aef5c2e84a276794e0c5bdd46450ce6cb93aca4f
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# spatial-neighborhood-statistics-computation

## Summary

Computes pixel- and neighborhood-based summary statistics — GLCM texture indices (contrast-group: contrast, dissimilarity, homogeneity; orderliness-group: entropy) — from temporal stacks of Sentinel-1 VH/VV gamma_0 backscatter to quantify horizontal structural heterogeneity of forest habitats.

## When to use

Use when you have terrain-corrected Sentinel-1 gamma_0 raster stacks (10 m pixels) and need spatial texture variables representing habitat structural heterogeneity for biodiversity models.

## When NOT to use

- Input rasters are raw GRDH digital numbers in slant-range coordinates rather than terrain-corrected gamma_0 — run the SNAP/GPT pre-processing chain first
- The analysis goal is vertical canopy structure (e.g. understorey or canopy height), where C-band Sentinel-1 has weak permeability and ALS is more appropriate
- A spatial speckle filter is explicitly required before texture analysis — this workflow relies on temporal (multi-acquisition median) speckle smoothing instead, to avoid loss of geometric detail

## Inputs

- Temporal stack of VH gamma_0 GeoTIFF rasters (e.g. ALB_gamma0_VH.tif) at 10 m pixel spacing
- Temporal stack of VV gamma_0 GeoTIFF rasters (e.g. ALB_gamma0_VV.tif) at 10 m pixel spacing
- Sentinel-1 acquisition dates table (e.g. ALB_acquisition_dates.csv)

## Outputs

- GLCM contrast-group texture index GeoTIFF layers (contrast, dissimilarity, homogeneity) per polarization
- GLCM orderliness-group texture index GeoTIFF layers per polarization
- Plot-level mean statistics (e.g. S1_mean.csv) for 1-ha sample plots

## How to apply

Load the temporal stacks of VH and VV gamma_0 rasters (e.g. ALB_gamma0_VH.tif and ALB_gamma0_VV.tif) in R using the raster package. Run the glcm package on each gamma_0 stack, computing grey-level co-occurrence textures with 32 grey levels and a 9x9-pixel moving window. Extract the contrast-group GLCM indices (contrast, dissimilarity, homogeneity) and the orderliness-group indices for both polarizations. Write the resulting texture layers to GeoTIFF rasters, preserving the 10 m pixel grid of the terrain-corrected inputs; temporal median backscatter over multiple acquisitions implicitly smooths speckle in time rather than in space, so no spatial speckle filter is applied before texture computation. Validate that every expected GLCM index layer exists for VH and VV and that each output raster matches the extent and resolution of its input gamma_0 stack.

## Related tools

- **R** (Environment in which the pixel- and neighborhood-based summary statistics are computed) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **glcm** (Computes grey-level co-occurrence texture measures (32 grey levels, 9x9-pixel window) on the gamma_0 stacks) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **raster** (Loads the VH/VV gamma_0 temporal stacks and writes the GLCM texture layers to GeoTIFF on the original 10 m grid) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgdal** (Supports reading/writing georeferenced raster and shapefile data within the R workflow) — github.com/So-YeonBae/Sentinel1-Biodiversity
- **rgeos** (Supports the required minimum R package set for the spatial statistics workflow) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
source("Sentinel1_pixel_neighborhood_statistics.r")  # R: runs glcm (32 grey levels, 9x9 window) on ALB_gamma0_VH.tif / ALB_gamma0_VV.tif stacks
```

## Evaluation signals

- Every expected GLCM index layer (contrast, dissimilarity, homogeneity, orderliness-group) exists for both VH and VV polarizations
- Each output texture raster matches the extent and 10 m resolution of the corresponding input gamma_0 stack
- Texture layers were computed with 32 grey levels and a 9x9-pixel moving window as specified
- Extracted plot means (S1_mean.csv-style output) align with the 1-ha plot shapefile locations (ALB_example_plots.*)
- Texture measures show expected correlations with radar structural-heterogeneity gradients (e.g. the third radar canonical axis) when coupled to downstream models

## Limitations

- C-band Sentinel-1 cannot well characterize the understorey due to weak permeability through overstorey layers
- The texture measures summarize horizontal (spatial) heterogeneity only; species richness and phylogenetic diversity of groups driven by understorey development were poorly predicted
- Outputs inherit geolocation and radiometric quality of the terrain-corrected gamma_0 inputs, which depend on precise orbits and SRTM 1 sec DEM during pre-processing

## Evidence

- [readme] GLCM over gamma_0 stacks with 32 grey levels and 9x9 window: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [other] Texture measures quantify spatial heterogeneity: [evidence span withheld — anchor status: not_found]
- [other] Texture measures correlate with structural heterogeneity axis: [evidence span withheld — anchor status: not_found]
- [readme] Speckle smoothing implicit in time, not space: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in space, thus not causing any further loss of geometric detail"
- [readme] Terrain-corrected gamma_0 at 10 m pixel spacing: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
