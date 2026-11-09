---
name: taxon-level-differential-abundance-analysis
description: Use when comparing taxonomic relative abundance between two treatment
  groups (e.g., control vs tylosin) from a processed 16S feature table, especially
  when Sex or other covariates must be adjusted and FDR-controlled taxon lists are
  required.
when_to_use_negative:
- The goal involves survival/time-to-event responses rather than a treatment-group
  comparison of microbial composition (use Module 3 or 4 instead, since Module 2 does
  not involve survival data)
- Input is raw 16S sequence data rather than a processed feature table — run the MiSurv
  data processing module (quality control, OTU picking at 97% similarity, rarefaction)
  first
- Only a single group is available or no treatment-group contrast can be defined in
  the metadata
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MiSurv
  role: Web cloud platform / local Shiny app providing the Data Processing module
    (CLR transformation) and Module 2 for covariate-adjusted comparative microbial
    composition analysis
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package underlying the phyloseq-format input object (biom.Rdata) and MiSurv's
    data processing/transformation steps
  repo: https://github.com/wg99526/MiSurvGit
- name: R
  role: Local runtime required to run the MiSurv Shiny implementation and the CLR-based
    linear regression workflow
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:a71031b122b6cfbdf5d0f860cb4320cd695af3da8673ca37f8fc88726c776a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - phyloseq
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# taxon-level-differential-abundance-analysis

## Summary

Performs covariate-adjusted differential abundance testing on CLR-transformed taxonomic abundances (phylum to genus) between treatment groups using MiSurv's Module 2, controlling the false discovery rate with Benjamini-Hochberg at 5%. Reproduces the MiSurv paper's finding of 66 significantly different taxa between control and tylosin groups.

## When to use

Use when comparing taxonomic relative abundance between two treatment groups (e.g., control vs tylosin) from a processed 16S feature table, especially when Sex or other covariates must be adjusted and FDR-controlled taxon lists are required.

## When NOT to use

- The goal involves survival/time-to-event responses rather than a treatment-group comparison of microbial composition (use Module 3 or 4 instead, since Module 2 does not involve survival data)
- Input is raw 16S sequence data rather than a processed feature table — run the MiSurv data processing module (quality control, OTU picking at 97% similarity, rarefaction) first
- Only a single group is available or no treatment-group contrast can be defined in the metadata

## Inputs

- Processed OTU count table (otu.tab.txt)
- Sample metadata table (sam.dat.txt) with treatment group and Sex columns
- phyloseq object (biom.Rdata) as alternative combined input
- Taxonomic annotation table (tax.tab.txt) defining phylum-to-genus ranks

## Outputs

- significant_taxa_fdr5.tsv: table of significant taxa (adjusted P < 0.05) with Benjamini-Hochberg adjusted P-values
- Count of significant taxa (expected: 66) from phylum to genus level

## How to apply

Load the processed OTU count table (otu.tab.txt) and sample metadata (sam.dat.txt) from the MiSurvGit Data folder into R (>4.2.0), optionally via the phyloseq object biom.Rdata. Apply the centered log-ratio (CLR) transformation, one of the data transformations offered by MiSurv's Data Processing module (alongside rarefaction, proportion, and arcsine square root), to obtain compositional normalized taxon abundances. For each taxon from phylum to genus, fit a linear regression of CLR-transformed abundance on treatment group adjusted for Sex via Module 2, which compares microbial composition between treatment groups with or without covariate adjustment. Collect raw P-values for the group coefficient across all taxa and apply Benjamini-Hochberg FDR correction at 5%; retain taxa with adjusted P < 0.05 and write them with adjusted P-values to significant_taxa_fdr5.tsv. The rationale is that CLR transformation addresses the compositional nature of amplicon count data while linear regression with covariates isolates the treatment effect.

## Related tools

- **MiSurv** (Web cloud platform / local Shiny app providing the Data Processing module (CLR transformation) and Module 2 for covariate-adjusted comparative microbial composition analysis) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package underlying the phyloseq-format input object (biom.Rdata) and MiSurv's data processing/transformation steps) — https://github.com/wg99526/MiSurvGit
- **R** (Local runtime required to run the MiSurv Shiny implementation and the CLR-based linear regression workflow) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- significant_taxa_fdr5.tsv contains exactly 66 significant taxa rows at BH FDR 5%
- Significant taxa span taxonomic ranks from phylum to genus
- Adjusted P-values correspond to the treatment-group coefficient of a Sex-adjusted linear regression on CLR-transformed abundances
- Result is consistent with the paper's reported findings in Figures 4 and 5
- Input files are the four MiSurvGit Data-folder files (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) or the equivalent biom.Rdata phyloseq object

## Limitations

- Local implementation requires a recent version of R (>4.2.0); no version numbers are given for R packages such as phyloseq
- Module 2 compares microbial composition between treatment groups and does not involve survival data
- Reference counts (e.g., 66 significant taxa) depend on the deposited example dataset (QIITA study 10508) and will not generalize to other inputs without recalibration

## Evidence

- [readme] Module 2 : Comparative analysis in microbial composition between treatment groups, not involving survival data, with or without covariate adjustment(s).: "Module 2 : Comparative analysis in microbial composition between treatment groups, not involving survival data, with or without covariate adjustment(s)."
- [readme] data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root): "data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root)"
- [discussion] with four individual files (see the files named "otu.tab.txt," "sam.dat.txt," "tax.tab.txt," and "tree.tre"): "with four individual files (see the files named "otu.tab.txt," "sam.dat.txt," "tax.tab.txt," and "tree.tre")"
- [readme] For local implementation using this GitHub repository, recent version of R (>4.2.0) is needed.: "For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."
