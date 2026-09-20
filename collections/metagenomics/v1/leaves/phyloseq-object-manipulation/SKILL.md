---
name: phyloseq-object-manipulation
description: Use when microbiome abundance, taxonomy, metadata, and/or tree data must
  be assembled into a phyloseq object (from biom.Rdata or otu.tab.txt/sam.dat.txt/tax.tab.txt/tree.tre)
  before rarefaction, diversity computation, or MiSurv Module 2-4 analysis.
when_to_use_negative:
- Running Module 1 (comparative survival analysis), which explicitly does not involve
  microbiome data and thus needs no phyloseq object
- Input is raw 16S rRNA reads that have not yet been processed by QIIME 2.0 into a
  feature table, taxonomy annotations, and phylogenetic tree
- Data supplied in a format outside MiSurv's accepted set (.rdata, .rds, .biom, .txt,
  .csv, .tsv, .tre) without prior conversion
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: phyloseq
  role: Provides the S4 object class and the rarefy_even_depth, estimate_richness,
    and distance functions used to manipulate the data and compute diversities
- name: picante
  role: Supplies the 'pd' function for phylogenetic diversity computed on the phyloseq
    object's tree
- name: MiSurv
  role: Web/Shiny platform whose Data Processing module ingests and transforms the
    phyloseq object before Modules 1-4
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:32aadc8ce072deaa6b1a5a9c4110d95a82256eeb546321915700f5c6e23bbc7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - QIIME2
  - R
  - phyloseq
  - picante
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# phyloseq-object-manipulation

## Summary

Construct, load, and manipulate 'phyloseq' S4 objects (OTU table, sample data, taxonomy table, phylogenetic tree) as the central data container for MiSurv's microbiome processing and its four analytic modules.

## When to use

Use when microbiome abundance, taxonomy, metadata, and/or tree data must be assembled into a phyloseq object (from biom.Rdata or otu.tab.txt/sam.dat.txt/tax.tab.txt/tree.tre) before rarefaction, diversity computation, or MiSurv Module 2-4 analysis.

## When NOT to use

- Running Module 1 (comparative survival analysis), which explicitly does not involve microbiome data and thus needs no phyloseq object
- Input is raw 16S rRNA reads that have not yet been processed by QIIME 2.0 into a feature table, taxonomy annotations, and phylogenetic tree
- Data supplied in a format outside MiSurv's accepted set (.rdata, .rds, .biom, .txt, .csv, .tsv, .tre) without prior conversion

## Inputs

- biom.Rdata (preassembled phyloseq object from the MiSurv Data folder)
- otu.tab.txt (tab-delimited OTU/feature count table)
- sam.dat.txt (sample metadata, e.g. treatment group, Sex, survival time)
- tax.tab.txt (taxonomy annotation table)
- tree.tre (phylogenetic tree file)

## Outputs

- phyloseq-class object with populated otu_table, sample_data, tax_table, and phy_tree slots
- rarefied feature table produced by rarefy_even_depth
- per-sample alpha-diversity index values (estimate_richness plus picante pd)
- beta-diversity distance matrices between samples

## How to apply

Load the preassembled phyloseq object 'biom.Rdata' from the MiSurv Data folder, or build the object from the four individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre'; local implementation requires R (>4.2.0). For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed. Once assembled, apply rarefaction via 'rarefy_even_depth' in the 'phyloseq' package to equalize library depths before diversity work. Compute the nine alpha-diversity indices per sample using 'estimate_richness' in the 'phyloseq' package plus phylogenetic diversity via the 'pd' function in the 'picante' package (the tree slot must be populated). Derive beta-diversity distance matrices with phyloseq's distance functions on the same processed table, then pass the object into MiSurv's Data Processing module (QC on kingdom, library size, mean proportion, taxonomic name) and Modules 2-4.

## Related tools

- **phyloseq** (Provides the S4 object class and the rarefy_even_depth, estimate_richness, and distance functions used to manipulate the data and compute diversities)
- **picante** (Supplies the 'pd' function for phylogenetic diversity computed on the phyloseq object's tree)
- **MiSurv** (Web/Shiny platform whose Data Processing module ingests and transforms the phyloseq object before Modules 1-4) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- Loaded/constructed object is of class 'phyloseq' with non-empty otu_table, sample_data, tax_table, and phy_tree slots
- Filtered tables match the deposited post-QC data: exactly 294 features across 172 subjects (7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species)
- After rarefy_even_depth, all retained samples share an equal, rarefied library depth
- estimate_richness yields the nine alpha-diversity indices and picante's pd returns phylogenetic diversity only when the tree slot is valid
- Results are reproducible from either biom.Rdata or the four individual files, matching the Data folder contents of the MiSurv GitHub repository

## Limitations

- Local implementation requires a recent version of R (>4.2.0); no version numbers are documented for phyloseq, picante, or other R packages
- Phylogenetic diversity via picante::pd fails or is undefined if tree.tre was not supplied or the tree slot is empty
- rarefy_even_depth discards reads from deeper-sequenced samples, so rarefied counts depend on the minimum library size (samples below 3,000 reads are dropped by default QC)

## Evidence

- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’): "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [discussion] with four individual files (see the files named "otu.tab.txt," "sam.dat.txt," "tax.tab.txt," and "tree.tre"): "with four individual files (see the files named "otu.tab.txt," "sam.dat.txt," "tax.tab.txt," and "tree.tre")"
- [discussion] the rarefaction (50) using the "rarefy_even_depth" in the "phyloseq" package: [evidence span withheld — anchor status: not_found]
- [discussion] using "estimate_richness" in the "phyloseq" package and "pd" function in the "picante" package: [evidence span withheld — anchor status: not_found]
- [readme] For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed.: "For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."
