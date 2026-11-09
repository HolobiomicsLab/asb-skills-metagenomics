---
name: beta-diversity-distance-analysis
description: Use when comparing gut microbial community composition between treatment
  groups (e.g., control vs tylosin) from a 16S feature table with taxonomic annotations
  and a phylogenetic tree, and you need distance matrices plus covariate-adjusted
  MiRKAT beta-diversity p-values.
when_to_use_negative:
- The analysis target is association between microbial composition and survival (time-to-event)
  responses rather than composition comparison between treatment groups — that is
  MiSurv Module 3, since Module 2 does not involve survival data.
- The goal is within-sample alpha-diversity (richness/diversity indices per sample)
  — that requires 'estimate_richness' in phyloseq and 'pd' in picante, not between-sample
  distance matrices.
- Input lacks a phylogenetic tree (no 'tree.tre' equivalent), precluding the phylogenetic
  beta-diversity distances used in the five-distance panel.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0610
tools:
- name: phyloseq
  role: Provides 'rarefy_even_depth' for rarefying the feature table and the distance
    functions used to compute the five beta-diversity distance matrices between samples.
- name: MiRKAT
  role: Kernel-based beta-diversity association testing of control-vs-tylosin group
    membership for each distance matrix, with covariate (Sex) adjustment.
- name: MiSurv
  role: Web cloud platform whose Module 2 performs comparative analysis in microbial
    composition between treatment groups with or without covariate adjustment(s);
    runnable locally from the GitHub repo with R >4.2.0 and shiny.
  repo: https://github.com/wg99526/MiSurvGit
- name: QIIME2
  role: Upstream pipeline that produced the feature table (97% similarity, Greengenes
    open-reference picking), taxonomic annotations, and phylogenetic tree that feed
    the beta-diversity computation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:9c0a9f93c6c0cd11bb7d74d276657193751f8034ca71336a6775b315c3f9d903
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiRKAT
  - MiSurv
  - QIIME2
  - R
  - phyloseq
  - picante
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# beta-diversity-distance-analysis

## Summary

Computes between-sample microbial community (beta-diversity) distance matrices from a rarefied phyloseq feature table and tests treatment-group composition differences, as implemented in MiSurv Module 2. Distances are quantified with phyloseq's distance functions and group association is assessed with MiRKAT kernel tests.

## When to use

Use when comparing gut microbial community composition between treatment groups (e.g., control vs tylosin) from a 16S feature table with taxonomic annotations and a phylogenetic tree, and you need distance matrices plus covariate-adjusted MiRKAT beta-diversity p-values.

## When NOT to use

- The analysis target is association between microbial composition and survival (time-to-event) responses rather than composition comparison between treatment groups — that is MiSurv Module 3, since Module 2 does not involve survival data.
- The goal is within-sample alpha-diversity (richness/diversity indices per sample) — that requires 'estimate_richness' in phyloseq and 'pd' in picante, not between-sample distance matrices.
- Input lacks a phylogenetic tree (no 'tree.tre' equivalent), precluding the phylogenetic beta-diversity distances used in the five-distance panel.

## Inputs

- phyloseq object 'biom.Rdata' (phyloseq format) from the MiSurv GitHub Data folder
- Individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre' as an alternative to biom.Rdata
- Treatment grouping variable (control vs tylosin) and Sex covariate from the sample metadata
- Raw 16S rRNA sequence data reference (QIITA study ID 10508) for provenance

## Outputs

- Five beta-diversity distance matrices between samples computed on the rarefied feature table
- MiRKAT kernel association p-values for control-vs-tylosin per distance
- Summary statistics table (CSV) assembling the five beta-diversity comparison results
- Beta-diversity group comparison figure reproducing Fig. 3B

## How to apply

Load the processed phyloseq object 'biom.Rdata' (or assemble it from 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the Data folder of the MiSurv GitHub repository, then rarefy the feature table using 'rarefy_even_depth' in the 'phyloseq' package so library-size differences do not bias distances. Compute the five beta-diversity distance matrices between samples with phyloseq's distance functions on the same rarefied table (phylogenetic and non-phylogenetic metrics both use the supplied tree from 'tree.tre'). For each distance, run the MiRKAT kernel association test of the control-vs-tylosin grouping with Sex as a covariate, per the paper's covariate-adjusted Module 2 design, and extract the kernel association p-values. Assemble the five distance results into a summary statistics CSV and render the beta-diversity group comparison figure reproducing Fig. 3B of Gu et al. 2023 (Microbiology Spectrum 11(3): e05059-22).

## Related tools

- **phyloseq** (Provides 'rarefy_even_depth' for rarefying the feature table and the distance functions used to compute the five beta-diversity distance matrices between samples.)
- **MiRKAT** (Kernel-based beta-diversity association testing of control-vs-tylosin group membership for each distance matrix, with covariate (Sex) adjustment.)
- **MiSurv** (Web cloud platform whose Module 2 performs comparative analysis in microbial composition between treatment groups with or without covariate adjustment(s); runnable locally from the GitHub repo with R >4.2.0 and shiny.) — https://github.com/wg99526/MiSurvGit
- **QIIME2** (Upstream pipeline that produced the feature table (97% similarity, Greengenes open-reference picking), taxonomic annotations, and phylogenetic tree that feed the beta-diversity computation.)

## Examples

```
library(phyloseq); ps <- readRDS("biom.Rdata"); ps <- rarefy_even_depth(ps); D <- distance(ps, method="bray")
```

## Evaluation signals

- All five beta-diversity distances must show statistically significant control-vs-tylosin differences by MiRKAT, matching the direction and significance reported in Fig. 3B of Gu et al. 2023 (Microbiology Spectrum 11(3): e05059-22).
- Distance matrices are computed on the rarefied table ('rarefy_even_depth' in phyloseq), not the raw feature table.
- The results CSV contains exactly five beta-diversity comparison rows, one per distance, each with a kernel association p-value.
- The rendered figure reproduces the layout of Fig. 3B (control vs tylosin beta-diversity comparison).
- The phyloseq object is constructed successfully either from biom.Rdata or from the four individual files, with sample IDs consistent between otu.tab.txt, sam.dat.txt, tax.tab.txt, and tree.tre.

## Limitations

- No version numbers are given for the R packages used (e.g., phyloseq, MiRKAT), so exact reproducibility of distance values may depend on package versions.
- The paper's demonstration is specific to the Zhang et al. T1D non-obese diabetic mouse gut dataset (QIITA study 10508, 16S rRNA V4, Greengenes 97% OTU picking); other data types may need re-parameterization.
- Local implementation requires a recent R version (>4.2.0) with shiny, and the README notes no changelog and no software license for MiSurv.

## Evidence

- [discussion] the rarefaction (50) using the "rarefy_even_depth" in the "phyloseq" package: [evidence span withheld — anchor status: not_found]
- [intro] PERMANOVA (23–25) and MiRKAT (26–28) for beta-diversity analysis: "PERMANOVA (23–25) and MiRKAT (26–28) for beta-diversity analysis"
- [intro] Module 2: Comparative analysis in microbial composition between treatment groups.: "Module 2: Comparative analysis in microbial composition between treatment groups."
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’): "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] Module 2 : Comparative analysis in microbial composition between treatment groups, not involving survival data, with or without covariate adjustment(s).: "**Module 2** : Comparative analysis in microbial composition between treatment groups, not involving survival data, with or without covariate adjustment(s)."
