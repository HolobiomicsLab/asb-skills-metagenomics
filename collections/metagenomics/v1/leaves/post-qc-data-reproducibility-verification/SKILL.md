---
name: post-qc-data-reproducibility-verification
description: Use when reproducing or auditing MiSurv's default QC pipeline on 16S
  OTU data and checking that filtered outputs match the deposited post-QC tables (expected
  294 features x 172 subjects).
when_to_use_negative:
- Input is already the post-QC biom.Rdata or post-QC otu.tab.txt/tax.tab.txt — use
  those directly instead of re-filtering
- Running Modules 1–4 (survival comparison, composition comparison, association testing,
  prediction) on already-QC'd data — this skill covers only the Data Processing QC
  step
- Shotgun metagenomics or data whose feature table did not originate from 16S rRNA
  V4 OTU picking at 97% similarity against Greengenes — the QC thresholds here were
  calibrated for that pipeline
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiSurv
  role: Source of the default QC criteria (kingdom, library size, mean proportion,
    taxonomic name) and of the example pre-/post-QC data files; can be run locally
    via the GitHub repository
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used to load the OTU/taxonomy tables and to filter taxa and samples
    during QC reconstruction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:d745da330c20c2ce21f3824f64fda1a5a2b10360d6314475c19bbec361d58aec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# post-qc-data-reproducibility-verification

## Summary

Reconstruct MiSurv's default microbiome quality control in R/phyloseq and verify that the resulting filtered abundance and taxonomy tables reproduce the deposited post-QC data exactly (294 features across 172 subjects in the example T1D gut microbiome dataset).

## When to use

Use when reproducing or auditing MiSurv's default QC pipeline on 16S OTU data and checking that filtered outputs match the deposited post-QC tables (expected 294 features x 172 subjects).

## When NOT to use

- Input is already the post-QC biom.Rdata or post-QC otu.tab.txt/tax.tab.txt — use those directly instead of re-filtering
- Running Modules 1–4 (survival comparison, composition comparison, association testing, prediction) on already-QC'd data — this skill covers only the Data Processing QC step
- Shotgun metagenomics or data whose feature table did not originate from 16S rRNA V4 OTU picking at 97% similarity against Greengenes — the QC thresholds here were calibrated for that pipeline

## Inputs

- Pre-QC OTU count table (otu.tab.txt, tab-delimited) from the MiSurv Data folder
- Taxonomy table (tax.tab.txt, tab-delimited) from the MiSurv Data folder
- Optional companion files: sam.dat.txt (sample data) and tree.tre (phylogenetic tree)
- Deposited post-QC phyloseq object (biom.Rdata) used as the reference for reproducibility comparison

## Outputs

- post_qc_otu_table.txt: filtered abundance table with exactly 294 features x 172 subjects
- post_qc_tax_table.txt: filtered taxonomy table for the 294 retained features
- Validation report confirming per-rank counts (7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species) and agreement with biom.Rdata

## How to apply

Load the pre-QC OTU count table (otu.tab.txt) and taxonomy table (tax.tab.txt) from the MiSurv Data folder into R using the phyloseq package. Apply the four default QC criteria in order: keep only taxa annotated with kingdom = Bacteria (removing erroneous or missing taxonomic names), drop samples whose total library size is below 3,000 reads, remove taxa whose mean relative proportion across retained samples is below 0.002%. Write the filtered abundance and taxonomy tables as tab-delimited text (post_qc_otu_table.txt, post_qc_tax_table.txt). Validate by asserting the filtered tables contain exactly 294 features and 172 subjects, checking the per-rank breakdown (7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species), and comparing them against the deposited post-QC files (biom.Rdata / otu.tab.txt / tax.tab.txt).

## Related tools

- **MiSurv** (Source of the default QC criteria (kingdom, library size, mean proportion, taxonomic name) and of the example pre-/post-QC data files; can be run locally via the GitHub repository) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used to load the OTU/taxonomy tables and to filter taxa and samples during QC reconstruction)

## Examples

```
Rscript -e 'otu <- read.delim("post_qc_otu_table.txt"); tax <- read.delim("post_qc_tax_table.txt"); stopifnot(nrow(otu) == 294, ncol(otu) == 172, nrow(tax) == 294)'
```

## Evaluation signals

- Filtered OTU table has exactly 294 features (rows) and 172 subjects (columns)
- Per-rank taxonomic counts match 7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species
- Filtered tables are identical (or equivalent under re-loading) to the deposited post-QC biom.Rdata / otu.tab.txt / tax.tab.txt
- No retained sample has a total library size below 3,000 reads; no retained taxon has a mean relative proportion below 0.002%
- Every retained taxon carries kingdom = Bacteria with a complete, non-erroneous taxonomic assignment

## Limitations

- These are MiSurv's default QC settings; changing any threshold (library size, mean proportion) will change the retained feature and subject counts
- No changelog or R package version numbers are provided, so exact package versions (e.g., phyloseq) needed for bitwise reproducibility are unknown
- The example validates against the Zhang et al. T1D NOD-mouse dataset (QIITA study 10508); other datasets will have different retained counts, so only the QC procedure — not the numbers — generalizes

## Evidence

- [readme] quality controls (kingdom, library size, mean proportion, taxonomic name): "(3) quality controls (kingdom, library size, mean proportion, taxonomic name)"
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format"
- [discussion] with four individual files (see the files named "otu.tab.txt," "sam.dat.txt," "tax.tab.txt," and "tree.tre"): "with four individual files (see the files named "otu.tab.txt," "sam.dat.txt," "tax.tab.txt," and "tree.tre")"
- [discussion] the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database: "the feature table using the 97% sequence similarity through open reference piking based on the Greengenes database"
- [discussion] deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508: "deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508"
