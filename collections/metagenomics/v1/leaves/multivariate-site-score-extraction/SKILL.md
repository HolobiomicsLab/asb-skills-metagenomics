---
name: multivariate-site-score-extraction
description: Use when ground-truth presence-absence community data (e.g. 463 monitoring
  plots across 12 taxa/functional groups) must be reduced to per-plot ordination axes
  (NMDS1/NMDS2) to model assemblage composition against remote-sensing predictors.
when_to_use_negative:
- The goal is modelling α-diversity (species richness) or phylogenetic diversity rather
  than assemblage composition (β-diversity) — those use different response variables
  in the paper.
- Only spectral or radar predictor stacks are available with no ground-truth species
  composition data — this skill derives responses from ground-truth biodiversity data,
  not Sentinel-1 gamma_0 statistics.
- Community data are abundance/continuous counts where Bray-Curtis on presence-absence
  would discard meaningful information; verify the presence-absence transformation
  is appropriate first.
tools:
- name: R
  role: Analysis language in which the presence-absence matrix is loaded, Bray-Curtis
    dissimilarities computed, and metaMDS ordinations run and scores extracted
- name: vegan
  role: R package providing metaMDS for the ordinations and dimcheckMDS for stress-based
    dimensionality selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:698f3eb5457638141fea87db351c876cc68ae24ce103c5f7b07a576a40f34f15
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
yes_rate: 0.75
n_claims: 4
---

# multivariate-site-score-extraction

## Summary

Quantifies species assemblage composition (β-diversity) per functional group as NMDS ordination site scores derived from presence-absence data, so the scores can serve as response variables in biodiversity models built from ALS and Sentinel-1 radar metrics.

## When to use

Use when ground-truth presence-absence community data (e.g. 463 monitoring plots across 12 taxa/functional groups) must be reduced to per-plot ordination axes (NMDS1/NMDS2) to model assemblage composition against remote-sensing predictors.

## When NOT to use

- The goal is modelling α-diversity (species richness) or phylogenetic diversity rather than assemblage composition (β-diversity) — those use different response variables in the paper.
- Only spectral or radar predictor stacks are available with no ground-truth species composition data — this skill derives responses from ground-truth biodiversity data, not Sentinel-1 gamma_0 statistics.
- Community data are abundance/continuous counts where Bray-Curtis on presence-absence would discard meaningful information; verify the presence-absence transformation is appropriate first.

## Inputs

- Presence-absence species composition matrix for 463 monitoring plots
- Functional-group assignments (twelve functional groups/taxa)
- R environment with the vegan package

## Outputs

- nmds_scores.csv: per-group, per-plot NMDS1/NMDS2 site scores
- Accepted ordination solutions (metaMDS objects) with stress values and chosen dimensionality per functional group

## How to apply

Load the presence-absence species composition matrix for the 463 monitoring plots together with the functional-group assignments in R. For each functional group, compute the Bray-Curtis dissimilarity matrix from the presence-absence data and run metaMDS from the vegan package on it. Select the dimensionality with dimcheckMDS, choosing the solution with the lowest stress meeting the stress < 0.2 criterion (dimension 2 for plants, lichens, moths, and carabids; dimension 3 for the remaining groups). Extract the NMDS1 and NMDS2 site scores for every plot from the accepted ordination, combine the per-group, per-plot scores into a single table, and write nmds_scores.csv. NMDS is used because the main axes of species assemblage composition for the twelve functional groups are quantified as non-metric multidimensional scaling ordination scores, which serve as response variables in the biodiversity models.

## Related tools

- **R** (Analysis language in which the presence-absence matrix is loaded, Bray-Curtis dissimilarities computed, and metaMDS ordinations run and scores extracted)
- **vegan** (R package providing metaMDS for the ordinations and dimcheckMDS for stress-based dimensionality selection)

## Examples

```
library(vegan); D <- vegdist(pa_matrix, method = "bray"); ord <- metaMDS(D, k = 2); sc <- scores(ord, sites = TRUE)
```

## Evaluation signals

- The accepted ordination for every functional group attains stress < 0.2 at the specified dimensionality (dimension 2 for plants, lichens, moths, carabids; dimension 3 for the remaining groups).
- nmds_scores.csv contains NMDS1 and NMDS2 scores for all 463 plots for every functional group.
- No missing/NA entries in the combined score table; row keys match the 463 plot identifiers and twelve functional-group labels.
- Each ordination is reproducible from the Bray-Curtis dissimilarity matrix of the presence-absence matrix via metaMDS (vegan) rather than from raw abundance data.
- Dimensionality choice is documented via dimcheckMDS output showing the lowest-stress solution meeting the criterion.

## Limitations

- Predictive power for assemblage composition of carabids, spiders and bats drops after accounting for regions, suggesting unmeasured regional determinants not captured by the ordination-plus-remote-sensing modelling.
- Study regions represent only the major temperate forest ecosystems of Central Europe, so ordination-score responses may not generalize to other forest systems.
- NMDS axis scores are only as good as the underlying ground-truth data; lack of georeferenced, well-stratified biodiversity data spanning multiple taxa is a major barrier to larger-scale application.

## Evidence

- [other] The main axes of species assemblage composition for the twelve functional groups were quantified as non-metric multidimensional scaling (NMDS) ordination scores: [evidence span withheld — anchor status: not_found]
- [other] Run metaMDS (vegan) on each dissimilarity matrix and use dimcheckMDS to select the dimensionality with the lowest stress meeting the stress < 0.2 criterion: [evidence span withheld — anchor status: not_found]
- [other] check that the accepted ordination for every functional group attains stress < 0.2 at the specified dimensionality and that NMDS1/NMDS2 scores are present for all 463 plots: [evidence span withheld — anchor status: not_found]
- [readme] Like the rest of the manuscript it was performed using R.: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
