---
name: bray-curtis-dissimilarity-computation
description: Use when you have a ground-truth presence-absence species composition
  matrix (plots × species) grouped into taxonomic/functional groups and need assemblage-composition
  response variables (β-diversity axes) for downstream modelling.
when_to_use_negative:
- The response variable is α-diversity (species richness) or phylogenetic diversity
  rather than assemblage composition — those were modelled separately in the paper
  and do not require NMDS ordination
- Input data are abundance counts or continuous environmental rasters rather than
  presence-absence species composition per plot — the Bray-Curtis computation as applied
  here expects presence-absence data
- Input is already a feature table of remote-sensing predictors (ALS or radar metrics)
  — this skill only builds the response variables, not the predictor set
tools:
- name: R
  role: Environment in which the dissimilarity matrices, metaMDS ordinations, and
    score tables are computed
- name: vegan
  role: Provides metaMDS for the NMDS ordination on Bray-Curtis dissimilarities and
    dimcheckMDS for stress-based dimensionality selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:7cd74dc22669b56beaf7f4e44d10fb525a5f2bb45b3699c082713cf55d7db6cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# bray-curtis-dissimilarity-computation

## Summary

Quantifies β-diversity of species assemblages by computing Bray-Curtis dissimilarity from a presence-absence composition matrix (463 monitoring plots × twelve functional groups) in R, then running non-metric multidimensional scaling (metaMDS in vegan) so that the resulting NMDS1/NMDS2 site scores can serve as response variables in biodiversity models.

## When to use

Use when you have a ground-truth presence-absence species composition matrix (plots × species) grouped into taxonomic/functional groups and need assemblage-composition response variables (β-diversity axes) for downstream modelling.

## When NOT to use

- The response variable is α-diversity (species richness) or phylogenetic diversity rather than assemblage composition — those were modelled separately in the paper and do not require NMDS ordination
- Input data are abundance counts or continuous environmental rasters rather than presence-absence species composition per plot — the Bray-Curtis computation as applied here expects presence-absence data
- Input is already a feature table of remote-sensing predictors (ALS or radar metrics) — this skill only builds the response variables, not the predictor set

## Inputs

- Presence-absence species composition matrix for 463 monitoring plots
- Functional-group assignment table mapping species to the twelve functional groups
- R environment with the vegan package installed

## Outputs

- Per-group Bray-Curtis dissimilarity matrices
- Per-group metaMDS ordination solutions (dimension 2 or 3 chosen by lowest stress < 0.2)
- nmds_scores.csv — combined NMDS1/NMDS2 site scores per plot per functional group for all 463 plots

## How to apply

Load the presence-absence species composition matrix for the 463 monitoring plots together with the functional-group assignments in R. For each functional group, compute the Bray-Curtis dissimilarity matrix from the presence-absence data. Run metaMDS (vegan) on each dissimilarity matrix and use dimcheckMDS to select the dimensionality with the lowest stress meeting the stress < 0.2 criterion — dimension 2 was accepted for plants, lichens, moths, and carabids, dimension 3 for the remaining groups. Extract the NMDS1 and NMDS2 site scores for every plot from the accepted ordination solution of each group and combine them into a single table (nmds_scores.csv). The rationale is that the first NMDS axes capture assemblage composition (β-diversity), which the paper then models against ALS and Sentinel-1 radar predictors.

## Related tools

- **R** (Environment in which the dissimilarity matrices, metaMDS ordinations, and score tables are computed)
- **vegan** (Provides metaMDS for the NMDS ordination on Bray-Curtis dissimilarities and dimcheckMDS for stress-based dimensionality selection)

## Examples

```
library(vegan); D <- vegdist(pa_matrix, method = "bray"); m <- metaMDS(D, k = 2); scores(m, display = "sites")
```

## Evaluation signals

- Every accepted ordination for the twelve functional groups attains stress < 0.2 at the specified dimensionality (k = 2 for plants, lichens, moths, carabids; k = 3 for the remaining groups)
- nmds_scores.csv contains NMDS1 and NMDS2 scores for all 463 plots for each functional group with no missing values
- Each dissimilarity matrix is square, symmetric, with zero diagonal and off-diagonal values in [0, 1] (Bray-Curtis on presence-absence data)
- dimcheckMDS stress-dimension curves show the accepted dimensionality has the lowest stress satisfying the < 0.2 criterion among candidates

## Limitations

- NMDS solutions depend on random starts; the accepted solution should be checked for stress < 0.2 at the specified dimensionality before extracting scores
- The compositional response variables are only as good as the ground-truth monitoring data — the paper notes that georeferenced, well-stratified biodiversity data spanning multiple taxa are a major barrier to larger-scale application
- Assemblage composition of carabids, spiders and bats showed a drop in predictive power after accounting for regions, reflecting geographical patterns or unmeasured environmental determinants not captured by the ordination itself

## Evidence

- [methods] Bray-Curtis dissimilarity from presence-absence data: [evidence span withheld — anchor status: not_found]
- [methods] metaMDS dimensionality selection criterion: [evidence span withheld — anchor status: not_found]
- [abstract] NMDS axes as assemblage-composition response variables in the biodiversity models: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [discussion] Data barrier for stratified multi-taxon species data: "The major barrier to the larger-scale application of our methodology is the lack of availability of georeferenced and well-stratified (both spatially and ecologically) biodiversity data that span"
