---
name: species-composition-nmds-ordination
description: Use when ground-truth biodiversity data are presence–absence species
  composition matrices for multiple taxa/plots and you need per-plot assemblage-composition
  axes (NMDS1/NMDS2) as model response variables.
when_to_use_negative:
- You already have fitted radar fixed-effects models and want predicted NMDS1 scores
  for new external plots — that is a prediction/external-validation step, not a new
  ordination
- Your response is α-diversity (species richness) or phylogenetic diversity rather
  than β-diversity assemblage composition
- The data are continuous abundance records rather than presence–absence composition
  matrices for the twelve functional groups
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3855
tools:
- name: R
  role: Environment in which the NMDS ordination and dissimilarity computations are
    performed
  repo: https://www.nature.com/articles/s41467-019-12737-x.pdf
- name: vegan
  role: Provides metaMDS for Bray–Curtis NMDS ordination and dimcheckMDS for dimensionality/stress
    checking
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:e40f56dbf1a8cb3d2b17f3755b7be283ceff5f652837011e81f845b88996b136
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rgdal
  - rgeos
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# species-composition-nmds-ordination

## Summary

Quantifies β-diversity assemblage composition by running NMDS ordination on presence–absence species matrices per functional group, producing NMDS1/NMDS2 site scores that serve as response variables in the ALS/radar biodiversity models.

## When to use

Use when ground-truth biodiversity data are presence–absence species composition matrices for multiple taxa/plots and you need per-plot assemblage-composition axes (NMDS1/NMDS2) as model response variables.

## When NOT to use

- You already have fitted radar fixed-effects models and want predicted NMDS1 scores for new external plots — that is a prediction/external-validation step, not a new ordination
- Your response is α-diversity (species richness) or phylogenetic diversity rather than β-diversity assemblage composition
- The data are continuous abundance records rather than presence–absence composition matrices for the twelve functional groups

## Inputs

- Presence–absence species composition matrix for the 463 monitoring plots
- Functional-group assignments for the twelve taxonomic/functional groups

## Outputs

- nmds_scores.csv: per-group, per-plot NMDS1 and NMDS2 site scores
- Accepted ordination solutions with stress values per functional group

## How to apply

Load the presence–absence species composition matrix for the 463 monitoring plots together with the functional-group assignments in R. For each of the twelve functional groups, compute the Bray–Curtis dissimilarity matrix from the presence–absence data, then run metaMDS (vegan). Use dimcheckMDS to select the dimensionality with the lowest stress meeting the stress < 0.2 criterion — dimension 2 for plants, lichens, moths, and carabids; dimension 3 for the remaining groups. Extract the NMDS1 and NMDS2 site scores for every plot from the accepted solution and combine them into a single per-group, per-plot table (nmds_scores.csv). The resulting first-axis scores (e.g. median radar-model R² of 0.57 across the twelve taxa for NMDS1) become the assemblage-composition responses modelled with ALS and Sentinel-1 radar metrics.

## Related tools

- **R** (Environment in which the NMDS ordination and dissimilarity computations are performed) — https://www.nature.com/articles/s41467-019-12737-x.pdf
- **vegan** (Provides metaMDS for Bray–Curtis NMDS ordination and dimcheckMDS for dimensionality/stress checking)

## Examples

```
Rscript -e 'library(vegan); m <- metaMDS(comp, distance="bray", k=3); dimcheckMDS(m)'
```

## Evaluation signals

- Every accepted ordination attains stress < 0.2 at the specified dimensionality (2 for plants, lichens, moths, carabids; 3 for the remaining groups)
- nmds_scores.csv contains NMDS1 and NMDS2 scores for all 463 plots for each of the twelve functional groups
- Dissimilarity matrices are Bray–Curtis computed from the presence–absence (binary) matrices, not abundance data
- Downstream sanity check: radar models fitted on the NMDS1 scores should reach median R² ≈ 0.57 over the twelve taxa, comparable to ALS (0.51)

## Limitations

- Dimensionality must be re-checked per group with dimcheckMDS; the stress < 0.2 criterion may not be met for new datasets, forcing a different k or rejecting the ordination
- NMDS scores are relative axes (sign/orientation arbitrary), so downstream external-validation comparisons (e.g. R² = 0.26 birds, 0.22 beetles) depend on consistent axis interpretation

## Evidence

- [abstract] NMDS response axes: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [methods] R used throughout: "Like the rest of the manuscript it was performed using R."
- [readme] R packages: "Minimum required packages which need to be installed in addition are: rgeos, raster, rgdal, glcm"
