---
name: canonical-correlation-analysis
description: Use when two sets of standardized continuous variables describing the
  same sampling units (e.g., radar-derived vs. ALS-derived forest structural metrics
  per 1-ha plot) must be compared via multivariate correlation to find significant
  shared axes.
when_to_use_negative:
- The goal is predicting biodiversity (species composition, richness, phylogenetic
  diversity) from single-remote-sensing predictors — use regression/NMDS modelling
  instead of cross-dataset CCoA
- The two variable sets are not measured on the same sampling units (plots), so canonical
  pairs would be uninterpretable
- Data are raw uncalibrated Sentinel-1 digital numbers rather than processed gamma_0
  summary statistics — complete the SNAP/GPT pre-processing and R summary-statistics
  workflow first
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0102
tools:
- name: R
  role: Statistical environment in which standardisation, cancor/candisc canonical
    correlation, axis significance testing, and extraction of structure coefficients
    are performed
  repo: https://cran.r-project.org/
- name: rgeos
  role: Minimum required R package installed alongside the workflow that produces
    the radar plot-mean metrics used in CCoA
- name: raster
  role: Minimum required R package for handling the gamma_0 temporal-stack rasters
    from which plot-mean radar metrics are derived prior to CCoA
- name: rgdal
  role: Minimum required R package for reading the plot shapefiles and rasters feeding
    the radar metric computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:a5d2edd4cbecb93a922f316ea0123ab36dd40b7d7fe26ee1fe2d5ebcb8dd31a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - raster
  - rgdal
  - rgeos
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# canonical-correlation-analysis

## Summary

Runs canonical correlation analysis (CCoA) in R between Sentinel-1 radar-derived plot-mean habitat metrics (from VH/VV gamma_0 temporal stacks) and ALS-derived structural metrics for the same 1-ha plots, to quantify how strongly the two remote-sensing datasets capture shared habitat-structure gradients and how many canonical axis pairs are statistically significant.

## When to use

Use when two sets of standardized continuous variables describing the same sampling units (e.g., radar-derived vs. ALS-derived forest structural metrics per 1-ha plot) must be compared via multivariate correlation to find significant shared axes.

## When NOT to use

- The goal is predicting biodiversity (species composition, richness, phylogenetic diversity) from single-remote-sensing predictors — use regression/NMDS modelling instead of cross-dataset CCoA
- The two variable sets are not measured on the same sampling units (plots), so canonical pairs would be uninterpretable
- Data are raw uncalibrated Sentinel-1 digital numbers rather than processed gamma_0 summary statistics — complete the SNAP/GPT pre-processing and R summary-statistics workflow first

## Inputs

- Radar-derived plot-mean structural metrics per 1-ha plot (S1_mean.csv, computed from VH and VV gamma_0 temporal stacks)
- ALS-derived structural metrics for the same 1-ha plots
- R environment with cancor/candisc capability (R, raster, rgdal, rgeos ecosystem)

## Outputs

- Canonical correlations for each retained axis pair
- Structure correlations (loadings) of the original radar and ALS variables on each canonical axis
- ccoa_canonical_structure.csv with canonical correlations and structure coefficients for the nine significant pairs

## How to apply

Load the radar-derived plot-mean structural metrics (e.g., from S1_mean.csv, computed over 1-ha plots from VH/VV gamma_0 temporal stacks) and the corresponding ALS-derived structural metrics for the same plots in R. Standardise both variable sets to zero mean and unit variance before analysis, since canonical correlation is sensitive to scale. Perform CCoA between the 13-variable radar set and the ALS set using cancor/candisc in R, then test the significance of each canonical axis pair (p < 0.05) and retain only the significant pairs — in this study nine of the pairs were significant. Extract each pair's canonical correlation and the structure correlations (loadings) of the original radar and ALS variables on each canonical axis, writing the retained pairs to ccoa_canonical_structure.csv. Interpret leading axes ecologically: the first aligns with a forest-maturity gradient and the second with structural heterogeneity.

## Related tools

- **R** (Statistical environment in which standardisation, cancor/candisc canonical correlation, axis significance testing, and extraction of structure coefficients are performed) — https://cran.r-project.org/
- **rgeos** (Minimum required R package installed alongside the workflow that produces the radar plot-mean metrics used in CCoA)
- **raster** (Minimum required R package for handling the gamma_0 temporal-stack rasters from which plot-mean radar metrics are derived prior to CCoA)
- **rgdal** (Minimum required R package for reading the plot shapefiles and rasters feeding the radar metric computation)

## Examples

```
Rscript -e "cc <- cancor(scale(radar_metrics), scale(als_metrics)); ccor <- cc$cor; write.csv(data.frame(pair=1:length(ccor), canonical_correlation=ccor), 'ccoa_canonical_structure.csv', row.names=FALSE)"
```

## Evaluation signals

- First canonical pair correlation is approximately 0.92 and the second approximately 0.75
- Nine canonical axis pairs are statistically significant at p < 0.05
- The nine significant pairs together account for ~96.30% of the variance of the two remote-sensing datasets
- Leading canonical axes align with the reported forest-maturity and structural-heterogeneity gradient interpretations
- Output file ccoa_canonical_structure.csv contains canonical correlations plus structure coefficients for exactly the retained (significant) pairs

## Limitations

- Radar sensitivity to habitat structure, while similar to ALS overall, poorly captures the understorey because C-band SAR has weak permeability through overstorey layers — CCoA loadings on understorey-driven metrics may be weak
- Results are grounded in five temperate forest regions of Central Europe; generality of the radar–ALS canonical structure in other forest ecosystems is untested
- Standardised metrics from S1_mean.csv were computed over 1-ha plots with temporal median backscatter; spatial speckle filtering was not applied since speckle is smoothed implicitly across the time stack

## Evidence

- [results] CCoA identified nine statistically significant canonical axis pairs (p < 0.05) between the 13 radar and ALS axes, together explaining 96.30% of the variance: [evidence span withheld — anchor status: not_found]
- [abstract] We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard: "We show that the sensitivity of radar to habitat structure is similar to that of airborne laser scanning (ALS), the current gold standard in the measurement of forest structure"
- [readme] Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
- [readme] Calculated mean values of all our 1-ha plots: S1_mean.csv: "4) Calculated mean values of all our 1-ha plots: S1_mean.csv"
- [discussion] Weak permeability through overstorey layers is unavoidable with space-borne C-band SAR systems, due to their short wavelengths: "Weak permeability through overstorey layers is unavoidable with space-borne C-band SAR systems, due to their short wavelengths21"
