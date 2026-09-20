---
name: multivariate-canonical-axis-interpretation
description: Use when two multivariate feature tables measured on the same plot/observation
  units (e.g. Sentinel-1 gamma_0 temporal-stack metrics vs ALS structural metrics)
  must be compared for shared variance and axis-level association.
when_to_use_negative:
- Input rows are species/biodiversity observations rather than aligned per-plot structural
  metric tables — biodiversity modelling in this study used NMDS and R² comparisons,
  not CCoA
- Sentinel-1 data are raw GRDH digital numbers rather than processed gamma_0 temporal
  stacks — apply the SNAP/GPT pre-processing workflow first (orbit, border/thermal
  noise, calibration, terrain flattening, terrain correction)
- The two variable sets are not measured on the same spatial units (e.g. mismatched
  plot extents or resolutions), so canonical pairs would be confounded by location
  rather than structure
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for standardising the radar and ALS metric sets and running canonical
    correlation analysis (cancor/candisc); the study states 'Like the rest of the
    manuscript it was performed using R.'
- name: raster
  role: R package used for handling the Sentinel-1 gamma_0 raster stacks from which
    the plot-mean structural metrics input to CCoA are derived
- name: rgdal
  role: Minimum required R package (with raster, rgeos, glcm) for reading plot shapefiles
    and rasters when deriving the summary statistics feeding CCoA
- name: rgeos
  role: Minimum required R package used for the spatial (plot/neighborhood) operations
    that produce the pixel- and neighborhood-based summary statistics prior to CCoA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:6bcc1975b1ff24bce548b1501415b1e4f9cbf94b488b39bffdbce440c1f87672
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# multivariate-canonical-axis-interpretation

## Summary

Perform and interpret canonical correlation analysis (CCoA) between two multivariate remote-sensing variable sets — in this study the 13 radar-derived (Sentinel-1 VH/VV gamma_0) plot-mean habitat metrics and the corresponding ALS-derived structural metrics — to quantify shared structural information and identify statistically significant canonical axis pairs.

## When to use

Use when two multivariate feature tables measured on the same plot/observation units (e.g. Sentinel-1 gamma_0 temporal-stack metrics vs ALS structural metrics) must be compared for shared variance and axis-level association.

## When NOT to use

- Input rows are species/biodiversity observations rather than aligned per-plot structural metric tables — biodiversity modelling in this study used NMDS and R² comparisons, not CCoA
- Sentinel-1 data are raw GRDH digital numbers rather than processed gamma_0 temporal stacks — apply the SNAP/GPT pre-processing workflow first (orbit, border/thermal noise, calibration, terrain flattening, terrain correction)
- The two variable sets are not measured on the same spatial units (e.g. mismatched plot extents or resolutions), so canonical pairs would be confounded by location rather than structure

## Inputs

- Radar-derived plot-mean structural metrics table (S1_mean.csv; 13 metrics computed over 1-ha plots from VH/VV gamma_0 temporal stacks)
- Corresponding ALS-derived structural metrics for the same 1-ha plots
- Plot identifier/geometry key aligning radar and ALS rows (e.g. ALB_example_plots shapefile)

## Outputs

- Canonical correlations for each significant axis pair (nine pairs, p < 0.05, first ~0.92, second ~0.75)
- Structure correlations (loadings) of original radar and ALS variables on each retained canonical axis (ccoa_canonical_structure.csv)
- Variance shared between the two remote-sensing datasets explained by the retained pairs (~96.30%)

## How to apply

Load the radar-derived plot-mean structural metrics (e.g. S1_mean.csv computed over 1-ha plots from VH/VV gamma_0 temporal stacks) and the ALS-derived metrics for the same plots into R. Standardise both variable sets to zero mean and unit variance prior to CCoA, then run canonical correlation with cancor/candisc between the radar metric set and the ALS metric set. Test the significance of each canonical axis pair (p < 0.05) and retain the significant pairs — the study retained nine significant pairs out of 13, explaining 96.30% of the variance of the two datasets. Extract the canonical correlation per pair (first pair ~0.92, second ~0.75) and the structure correlations (loadings) of the original radar and ALS variables on each retained axis, then interpret leading axes ecologically (e.g. forest-maturity and structural-heterogeneity gradients). This underpins the paper's claim that radar sensitivity to habitat structure is similar to ALS, the gold standard for forest structure measurement.

## Related tools

- **R** (Environment for standardising the radar and ALS metric sets and running canonical correlation analysis (cancor/candisc); the study states 'Like the rest of the manuscript it was performed using R.')
- **raster** (R package used for handling the Sentinel-1 gamma_0 raster stacks from which the plot-mean structural metrics input to CCoA are derived)
- **rgdal** (Minimum required R package (with raster, rgeos, glcm) for reading plot shapefiles and rasters when deriving the summary statistics feeding CCoA)
- **rgeos** (Minimum required R package used for the spatial (plot/neighborhood) operations that produce the pixel- and neighborhood-based summary statistics prior to CCoA)

## Examples

```
cc <- cancor(scale(radar_metrics), scale(als_metrics)); print(cc$cor)
```

## Evaluation signals

- First canonical pair correlation is approximately 0.92 and the second approximately 0.75
- Exactly nine canonical axis pairs are statistically significant at p < 0.05 out of the 13 tested
- The nine significant pairs jointly explain approximately 96.30% of the variance of the two remote-sensing datasets
- Structure coefficients show leading axes aligning with reported forest-maturity and structural-heterogeneity gradient interpretations
- Output table (ccoa_canonical_structure.csv) contains canonical correlations and structure coefficients for all retained pairs

## Limitations

- The comparison is grounded in five temperate forest regions of Central Europe; generality to other forest types was not tested
- Sentinel-1 C-band radar has weak permeability through overstorey layers, so structural metrics dominated by understorey development are poorly characterised
- Speckle is handled implicitly by temporal median backscatter over multiple acquisitions rather than spatial filtering, so single-acquisition inputs would require different treatment

## Evidence

- [readme] This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
- [abstract] We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard in the measurement of forest structure: "We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard in the measurement of forest structure"
- [abstract] median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
