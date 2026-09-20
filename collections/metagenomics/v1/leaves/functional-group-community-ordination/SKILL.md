---
name: functional-group-community-ordination
description: Use when presence-absence species composition data are available for
  multiple plots and per-plot community-composition response variables (e.g. NMDS1/NMDS2
  site scores) are needed for taxa or functional groups.
when_to_use_negative:
- When the target response variables are species richness or phylogenetic diversity
  (α-diversity measures) rather than assemblage composition (β-diversity).
- When abundance (count or cover) data are available and the analysis requires quantitative
  dissimilarity rather than presence-absence-based Bray-Curtis dissimilarity.
tools:
- name: R
  role: Environment in which the ordination workflow is performed, consistent with
    the rest of the manuscript's analysis being conducted in R.
  repo: https://cran.r-project.org/
- name: vegan
  role: Provides metaMDS for the NMDS ordinations on Bray-Curtis dissimilarity matrices
    and dimcheckMDS for stress-based dimensionality selection.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:0519dcdb15d5cd659d7f25a6babd1304c6c0492b472291a74faaaf2def271a09
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# functional-group-community-ordination

## Summary

Quantifies species assemblage composition (β-diversity) of the twelve functional groups as non-metric multidimensional scaling (NMDS) ordination scores computed from presence-absence ground-truth data, which then serve as response variables in the ALS- and radar-based biodiversity models.

## When to use

Use when presence-absence species composition data are available for multiple plots and per-plot community-composition response variables (e.g. NMDS1/NMDS2 site scores) are needed for taxa or functional groups.

## When NOT to use

- When the target response variables are species richness or phylogenetic diversity (α-diversity measures) rather than assemblage composition (β-diversity).
- When abundance (count or cover) data are available and the analysis requires quantitative dissimilarity rather than presence-absence-based Bray-Curtis dissimilarity.

## Inputs

- presence-absence species composition matrix for 463 monitoring plots (rows = plots, columns = species)
- functional-group assignments mapping each species to one of the twelve functional groups

## Outputs

- per-group, per-plot NMDS1 and NMDS2 site scores combined into a single table (nmds_scores.csv)
- per-group accepted ordination solutions with their stress values

## How to apply

Load the presence-absence species composition matrix for the 463 monitoring plots together with the functional-group assignments in R. For each of the twelve functional groups, compute a Bray-Curtis dissimilarity matrix from the presence-absence data, then run metaMDS (vegan) on each dissimilarity matrix. Use dimcheckMDS to select the dimensionality with the lowest stress meeting the stress < 0.2 criterion — dimension 2 for plants, lichens, moths, and carabids; dimension 3 for the remaining groups. Extract the NMDS1 and NMDS2 site scores for every plot from the accepted ordination solution of each group, since the first NMDS axes are the assemblage-composition variables that were modelled against Sentinel-1 radar and ALS metrics.

## Related tools

- **R** (Environment in which the ordination workflow is performed, consistent with the rest of the manuscript's analysis being conducted in R.) — https://cran.r-project.org/
- **vegan** (Provides metaMDS for the NMDS ordinations on Bray-Curtis dissimilarity matrices and dimcheckMDS for stress-based dimensionality selection.)

## Examples

```
library(vegan); d <- vegdecide(pa_matrix, method = "bray", binary = TRUE); nmds <- metaMDS(d, k = 2); scores(nmds, display = "sites")
```

## Evaluation signals

- The accepted ordination for every functional group attains stress < 0.2 at the specified dimensionality (2 for plants, lichens, moths, and carabids; 3 for the remaining groups).
- nmds_scores.csv contains NMDS1/NMDS2 scores for all 463 plots for every functional group.
- Dissimilarity matrices are computed with the Bray-Curtis index on binary (presence-absence) data, not abundance data.
- Each group's ordination is derived only from the species belonging to that functional group, yielding twelve independent ordination solutions.

## Limitations

- Stress < 0.2 was used as the acceptance criterion; groups requiring higher dimensionality (dimension 3) produce scores beyond the two extracted NMDS1/NMDS2 axes.
- Assemblage composition (β-diversity) for carabids, spiders and bats showed a drop in predictive power after accounting for regions, suggesting unmeasured regional determinants not captured by the ordination-based response variables.
- Scores are tied to the 463-plot monitoring design; ordinations are not directly transferable to new plots without reference to the original dissimilarity structure.

## Evidence

- [other] Bray-Curtis dissimilarity and metaMDS workflow: [evidence span withheld — anchor status: not_found]
- [other] Dimensionality selection via stress criterion: [evidence span withheld — anchor status: not_found]
- [abstract] First NMDS axes as modelled composition variables: "median R² over twelve taxa by ALS and radar are 0.51 and 0.57 respectively for the first non-metric multidimensional scaling axes representing assemblage composition"
- [readme] R as the analysis environment: "This section builds on temporal stacks of Sentinel-1 gamma_0 data processed as described above. Like the rest of the manuscript it was performed using R."
