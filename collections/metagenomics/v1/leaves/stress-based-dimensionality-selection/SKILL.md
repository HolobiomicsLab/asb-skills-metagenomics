---
name: stress-based-dimensionality-selection
description: Use when quantifying β-diversity (assemblage composition) from a presence-absence
  species matrix per functional group and you must decide how many NMDS dimensions
  to retain before extracting site scores as response variables.
when_to_use_negative:
- The response variables needed are α-diversity (species richness) or phylogenetic
  diversity measures rather than assemblage composition (β-diversity) ordination scores
- The composition data are already summarized (e.g. a dissimilarity matrix is provided)
  so dimensionality selection must be made on a different basis than recomputing Bray-Curtis
  from presence-absence data
- A stress value ≥ 0.2 persists at all candidate dimensionalities, in which case NMDS
  scores should not be accepted as response variables without revisiting the composition
  data
tools:
- name: vegan
  role: Provides metaMDS to fit the NMDS ordination on each Bray-Curtis dissimilarity
    matrix and dimcheckMDS to compare stress across dimensionalities for the stress
    < 0.2 criterion
- name: R
  role: Environment in which the composition matrices are loaded, dissimilarities
    computed, ordinations run, and nmds_scores.csv assembled
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:67da8fa711f834d3b0f4ecbd930a5edab574ff5b65ee0087a28a55a183b24bec
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
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# stress-based-dimensionality-selection

## Summary

Selects the dimensionality of non-metric multidimensional scaling (NMDS) ordinations of assemblage composition by choosing the solution with the lowest stress that meets the stress < 0.2 criterion, so that NMDS1/NMDS2 site scores are valid β-diversity response variables for biodiversity modelling.

## When to use

Use when quantifying β-diversity (assemblage composition) from a presence-absence species matrix per functional group and you must decide how many NMDS dimensions to retain before extracting site scores as response variables.

## When NOT to use

- The response variables needed are α-diversity (species richness) or phylogenetic diversity measures rather than assemblage composition (β-diversity) ordination scores
- The composition data are already summarized (e.g. a dissimilarity matrix is provided) so dimensionality selection must be made on a different basis than recomputing Bray-Curtis from presence-absence data
- A stress value ≥ 0.2 persists at all candidate dimensionalities, in which case NMDS scores should not be accepted as response variables without revisiting the composition data

## Inputs

- Presence-absence species composition matrix for 463 monitoring plots (R data frame / CSV)
- Functional-group assignments mapping each taxon to one of twelve functional groups

## Outputs

- Accepted metaMDS ordination solution per functional group (dimension 2 or 3, stress < 0.2)
- nmds_scores.csv — per-group, per-plot NMDS1 and NMDS2 site scores for all 463 plots

## How to apply

For each of the twelve functional groups, compute a Bray-Curtis dissimilarity matrix from the 463-plot presence-absence composition matrix in R, then run metaMDS from the vegan package on each dissimilarity matrix. Use dimcheckMDS to inspect stress across candidate dimensionalities and accept the solution with the lowest stress that satisfies the stress < 0.2 criterion; in this study this gave dimension 2 for plants, lichens, moths, and carabids, and dimension 3 for the remaining groups. Extract NMDS1 and NMDS2 site scores for every plot from each accepted ordination, combine them per group and plot into a single table (nmds_scores.csv), and use the first NMDS axes as the assemblage-composition response variables modelled against ALS and Sentinel-1 radar metrics. The rationale is that a stress below 0.2 indicates the ordination gives a usable low-dimensional representation of Bray-Curtis structure, avoiding both over- and under-compressed assemblage gradients.

## Related tools

- **vegan** (Provides metaMDS to fit the NMDS ordination on each Bray-Curtis dissimilarity matrix and dimcheckMDS to compare stress across dimensionalities for the stress < 0.2 criterion)
- **R** (Environment in which the composition matrices are loaded, dissimilarities computed, ordinations run, and nmds_scores.csv assembled)

## Examples

```
library(vegan); ord <- metaDMS <- metaMDS(spp_matrix, distance = "bray", k = 2); dimcheckMDS(spp_matrix)
```

## Evaluation signals

- The accepted ordination for every one of the twelve functional groups attains stress < 0.2 at its specified dimensionality (dimension 2 for plants, lichens, moths, carabids; dimension 3 for the others)
- nmds_scores.csv contains NMDS1 and NMDS2 values for every plot, i.e. complete coverage of all 463 plots across all groups
- dimcheckMDS output shows the chosen dimension has the lowest stress among candidates meeting the 0.2 threshold (no higher-dimension solution with materially lower stress was skipped)
- The NMDS1 scores downstream reproduce the paper's reported model performance (median R² of 0.51 for ALS and 0.57 for radar over twelve taxa on the first NMDS axes)

## Limitations

- Stress < 0.2 is a heuristic acceptability threshold; dimensionality choices (2 vs 3) are group-specific and were fixed in this study, not re-tuned
- The skill only addresses the β-diversity (assemblage composition) component — Sentinel-1 radar in this study showed shortcomings for species richness and phylogenetic diversity of groups driven by understorey development, which NMDS scores alone do not fix
- Validity depends on the quality and stratification of the underlying ground-truth biodiversity data; georeferenced, well-stratified multi-taxa data are a known barrier to scaling this approach

## Evidence

- [methods] Run metaMDS (vegan) on each dissimilarity matrix and use dimcheckMDS to select the dimensionality with the lowest stress meeting the stress < 0.2 criterion: [evidence span withheld — anchor status: not_found]
- [abstract] median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [results] The main axes of species assemblage composition for the twelve functional groups were quantified as non-metric multidimensional scaling (NMDS) ordination scores, which served as response variables: [evidence span withheld — anchor status: not_found]
