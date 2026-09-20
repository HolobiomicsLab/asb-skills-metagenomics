---
name: phylogenetic-diversity-calculation
description: Use when you have an OTU/species count table per sample plus a phylogenetic
  tree and need per-sample phylogenetic α-diversity indices (PD, PE, PQE, or richness/Shannon/Simpson)
  as regression outcomes for downstream meta-analysis.
when_to_use_negative:
- Input is raw, un-rarefied sequence data — the article rarefies the OTU table to
  an even level of total reads per sample before computing diversity, so diversity
  computed on uneven depths is not comparable
- You only need taxonomic (non-phylogenetic) indices and have no phylogenetic tree
  — PD/PE/PQE require branch lengths and the tree
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0610
tools:
- name: vegan
  role: R package used to compute the α-diversity indices (including phylogenetic
    PD/PE/PQE) from the count tables and phylogenetic tree; imported by the mMeta
    package
  repo: https://github.com/hk1785/mMeta
- name: PyNAST
  role: Used to construct the OTU table and phylogenetic tree per study that are the
    input to the diversity calculation
- name: Resphera Insight
  role: Used with PyNAST to construct the OTU table and phylogenetic tree feeding
    the diversity computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:82e04c21b6559fee0c2da7e612e490c2ce1d67e8c37357fa87d8ff9aeaddbeda
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - PyNAST
  - R
  - R package mMeta
  - Resphera Insight
  - vegan
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# phylogenetic-diversity-calculation

## Summary

Compute phylogenetically weighted α-diversity indices (PD, PE, PQE) for each sample from a rarefied OTU count table and its phylogenetic tree, in R using the vegan package, so that study-specific effect estimates for phylogenetic diversity can feed downstream meta-analysis (mMeta/aMeta).

## When to use

Use when you have an OTU/species count table per sample plus a phylogenetic tree and need per-sample phylogenetic α-diversity indices (PD, PE, PQE, or richness/Shannon/Simpson) as regression outcomes for downstream meta-analysis.

## When NOT to use

- Input is raw, un-rarefied sequence data — the article rarefies the OTU table to an even level of total reads per sample before computing diversity, so diversity computed on uneven depths is not comparable
- You only need taxonomic (non-phylogenetic) indices and have no phylogenetic tree — PD/PE/PQE require branch lengths and the tree

## Inputs

- Rarefied OTU count table (samples x species/OTUs) per study
- Phylogenetic tree with branch lengths covering the OTUs
- Sample metadata / covariates (e.g., HIV status, x1, x2) for the regression step

## Outputs

- Per-sample α-diversity values for each index: Species richness, Shannon, Simpson, PD, PE, PQE
- Study-specific effect estimates (regression coefficients) and standard errors per α-diversity index for meta-analysis input

## How to apply

Construct the OTU table and phylogenetic tree per study (the paper used Resphera Insight and PyNAST, then rarefied the OTU table to an even level of total reads per sample to control for differing sequencing depth). In R, load the tree and the rarefied count table and compute the six α-diversity indices — Species richness, Shannon, Simpson, PD, PE and PQE — for each sample, with the phylogenetic indices (PD, PE, PQE) incorporating branch lengths from the tree. In the simulation setting, the paper drew 100 OTUs and their phylogenetic tree from the respiratory-tract microbiome data as surrogates and used vegan for the diversity computation. Each per-sample index then becomes the outcome y in a per-study least-squares regression (e.g., y ~ HIV status or the linear model with covariates x1 and x2), yielding the study-specific effect estimate and standard error per index that mMeta.aMeta consumes. Pairing PD with non-phylogenetic indices (Richness, Shannon, Simpson) lets the multi-marker meta-analysis make a global 'α-diversity' conclusion.

## Related tools

- **vegan** (R package used to compute the α-diversity indices (including phylogenetic PD/PE/PQE) from the count tables and phylogenetic tree; imported by the mMeta package) — https://github.com/hk1785/mMeta
- **PyNAST** (Used to construct the OTU table and phylogenetic tree per study that are the input to the diversity calculation)
- **Resphera Insight** (Used with PyNAST to construct the OTU table and phylogenetic tree feeding the diversity computation)

## Examples

```
library(vegan); data(HIV.Sum.Data); est <- HIV.Sum.Data$est; std.err <- HIV.Sum.Data$std.err; out <- mMeta.aMeta(est, std.err, seed=123)
```

## Evaluation signals

- All six indices (Species richness, Shannon, Simpson, PD, PE, PQE) are produced for every sample in every study from the rarefied count table and tree
- Diversity is computed on the rarefied OTU table (even total reads per sample), not on raw uneven-depth counts
- In the simulation, the tree and 100 OTUs trace back to the respiratory-tract microbiome data used as surrogates for 100 microbial species
- Downstream regression per index yields a coefficient and SE per study per marker, matching the schema required by mMeta.aMeta (est/std.err data.frames with matched NA locations, ≥10 studies)

## Limitations

- The six α-diversity indices are not always available from prior studies, while other indices may be available; the choice of indices constrains which summary data can be meta-analyzed
- Rarefaction to an even total reads per sample is required for comparability; diversity on uneven depths controls poorly for differing total reads
- No version number is stated for the mMeta R package that imports vegan for this computation, and no changelog was found for the repository

## Evidence
