---
name: alpha-diversity-index-calculation
description: Use when you have a rarefied OTU count table per study plus a phylogenetic
  tree and need per-sample α-diversity summaries (richness, Shannon, Simpson, PD,
  PE, PQE) to feed downstream regression or meta-analysis.
when_to_use_negative:
- Input is already per-marker effect estimates and standard errors (Sum.Data-style
  summary statistics) — skip directly to meta-analysis with mMeta.aMeta
- Study count tables are not rarefied to an even total-read level — index comparisons
  across samples will be biased by sequencing depth
- The marker set has no phylogenetic tree available — phylogenetic indices PD, PE,
  and PQE cannot be computed
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: vegan
  role: R package used for computing α-diversity indices from count tables (imports
    of mMeta); a prerequisite install for the pipeline
  repo: https://github.com/hk1785/mMeta
- name: R package mMeta
  role: Provides simulation utilities and the downstream meta-analysis consuming α-diversity-derived
    summary statistics
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment in which rarefaction and diversity index computation are
    implemented
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:a634a40bef7086d37053ad5c5357df847aa6ea9e0ff31bdb3e1e35baeac92dff
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  - vegan
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# alpha-diversity-index-calculation

## Summary

Compute per-sample α-diversity indices — Species richness (Observed), Shannon, Simpson, and phylogeny-aware PD, PE, PQE — from an OTU table and phylogenetic tree, after rarefying the OTU table to an even level of total reads per sample, as done for microbiome studies in the mMeta/aMeta meta-analysis framework.

## When to use

Use when you have a rarefied OTU count table per study plus a phylogenetic tree and need per-sample α-diversity summaries (richness, Shannon, Simpson, PD, PE, PQE) to feed downstream regression or meta-analysis.

## When NOT to use

- Input is already per-marker effect estimates and standard errors (Sum.Data-style summary statistics) — skip directly to meta-analysis with mMeta.aMeta
- Study count tables are not rarefied to an even total-read level — index comparisons across samples will be biased by sequencing depth
- The marker set has no phylogenetic tree available — phylogenetic indices PD, PE, and PQE cannot be computed

## Inputs

- OTU count table (samples x species) per study
- Phylogenetic tree of OTUs (e.g., 100 common respiratory-tract microbiome OTUs)
- Rarefied OTU table at an even level of total reads per sample

## Outputs

- Per-sample values of six α-diversity indices: Species richness (Observed), Shannon, Simpson, PD, PE, PQE
- Study-level summary statistics (regression coefficients and standard errors) per α-diversity index for downstream meta-analysis

## How to apply

First rarefy each study's OTU table to an even level of total reads per sample to control for differing sequencing depth. Then compute the six α-diversity indices for each sample in each study: three non-phylogenetic indices (Species richness/Observed, Shannon, Simpson) and three phylogenetic indices (PD, PE, PQE) using the OTU table and phylogenetic tree. In R, vegan is imported by the mMeta package for diversity computation; the mMeta package installs via devtools::install_github('hk1785/mMeta', force=T). In simulations, generate Dirichlet-multinomial count tables from the 100 respiratory-tract OTUs and their tree (per-study sample sizes 50–100, total reads 1,000–5,000) and compute the same six indices per sample. The resulting per-sample α-diversity values become the response in per-study regression models (linear regression adjusting for covariates x1 and x2, or simple logistic regression by HIV status) whose coefficients and standard errors are the summary statistics that mMeta and aMeta consume.

## Related tools

- **vegan** (R package used for computing α-diversity indices from count tables (imports of mMeta); a prerequisite install for the pipeline) — https://github.com/hk1785/mMeta
- **R package mMeta** (Provides simulation utilities and the downstream meta-analysis consuming α-diversity-derived summary statistics) — https://github.com/hk1785/mMeta
- **R** (Runtime environment in which rarefaction and diversity index computation are implemented)

## Examples

```
library(mMeta); library(vegan)  # compute vegan diversity indices on rarefied OTU table, then feed est/std.err to mMeta.aMeta(est, std.err, seed=123)
```

## Evaluation signals

- Exactly six indices per sample: Species richness (Observed), Shannon, Simpson, PD, PE, PQE — no others
- OTU table is rarefied to an even level of total reads per sample before index computation
- Phylogenetic indices (PD, PE, PQE) require the phylogenetic tree; non-phylogenetic ones do not
- Per-study regression of each index (adjusting for x1 Bernoulli and x2 N(0,1)) yields effect estimates and standard errors matched in shape for mMeta/aMeta input (missing entries must align between est and std.err)
- In simulation runs, count tables derive from the 100 respiratory-tract OTUs and their tree with sample sizes from discrete uniform 50–100 and total reads 1,000–5,000

## Limitations

- The six α-diversity indices are not always available from prior studies while other indices may be available, limiting comparability across meta-analyzed studies
- Rarefaction discards reads to equalize totals per sample, which reduces effective depth for low-read samples
- Phylogenetic indices require a tree; studies without phylogenies contribute only the three non-phylogenetic indices

## Evidence

- [methods] We rarefied the OTU table for each study to an even level of total reads per sample: "We rarefied the OTU table for each study to an even level of total reads per sample, meant to control for differing total reads per sample"
- [readme] Six alpha-diversity indices for the HIV summary data: "A list of regression coefficients and their standard errors for the 15 human microbiome studies for the association between HIV status and each of the 6 alpha-diversity indices, Richness (Observed),"
- [methods] Simulation grounding of OTU set and tree: "we employed the 100 common operational taxonomic units (OTUs) and their phylogenetic tree in the respiratory-tract microbiome data33 as surrogates for 100 microbial species"
- [methods] OTU table and tree construction upstream of index computation: "the OTU table and the phylogenetic tree for each study were constructed using Resphera Insight and PyNAST53"
- [methods] Regression step consuming the indices: "We estimated the regression coefficients and their standard errors for each α-diversity index in each study based on the least squares estimation adjusting for x1k and x2k"
