---
name: rnaseq-preprocessing
description: Use when per-sample HTSeq gene count files or Cufflinks-style gene FPKM
  files must be merged into one CSV count/FPKM matrix so downstream bulk RNA-seq apps
  (e.g., DESeq2Shiny) can consume them.
when_to_use_negative:
- Input is already a single merged feature-by-sample count matrix ready for DESeq2Shiny
- Gene identifier columns differ across the input files — GeneCountMerger explicitly
  will not work in that case
- Task is single-cell scRNA-seq preprocessing (filtering, normalization, PCA, UMAP/t-SNE),
  which belongs to Seurat-based NASQAR apps instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GeneCountMerger
  role: NASQAR Shiny preprocessing app that merges individual HTSeq-style gene count
    files into one matrix
  repo: https://github.com/nasqar/GeneCountMerger
- name: NASQAR
  role: Web platform hosting the preprocessing app so non-programming experts can
    run it without coding
  repo: https://github.com/nasqar/NASQAR
- name: R
  role: Implementation language; dplyr-style full_join operations assemble the merged
    genes-by-samples matrix
- name: DESeq2Shiny
  role: Downstream bulk RNA-seq app that directly consumes the merged CSV count/FPKM
    matrix
  repo: https://github.com/nasqar/deseq2shiny
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:7ddc8fbe99f9771c8299f91b72aa985df8bdc42b849710f7c4f3f2852dd67c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/GeneCountMerger
  tools:
  - DESeq2Shiny
  - GeneCountMerger
  - NASQAR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# rnaseq-preprocessing

## Summary

NASQAR-style preprocessing of bulk RNA-seq inputs: merging individual per-sample gene count files (e.g., HTSeq output) or Cufflinks-style FPKM files into a single genes-by-samples CSV matrix ready for downstream apps such as DESeq2Shiny.

## When to use

Use when per-sample HTSeq gene count files or Cufflinks-style gene FPKM files must be merged into one CSV count/FPKM matrix so downstream bulk RNA-seq apps (e.g., DESeq2Shiny) can consume them.

## When NOT to use

- Input is already a single merged feature-by-sample count matrix ready for DESeq2Shiny
- Gene identifier columns differ across the input files — GeneCountMerger explicitly will not work in that case
- Task is single-cell scRNA-seq preprocessing (filtering, normalization, PCA, UMAP/t-SNE), which belongs to Seurat-based NASQAR apps instead

## Inputs

- Per-sample gene count files (e.g., HTSeq-style two-column output: gene identifier, raw count)
- Per-sample Cufflinks-style tabular gene FPKM/abundance files
- Gene ID-to-name mapping (optional, for gene name conversion)

## Outputs

- merged_gene_counts.csv — a gene-by-sample matrix of non-negative integer counts
- merged_fpkm_matrix.csv — a genes-by-samples CSV matrix of FPKM values with the gene identifier as the first column

## How to apply

Load each per-sample two-column table (gene identifier, raw count or FPKM) with R's readr/read.table and key rows on the gene identifier column. For count data, drop HTSeq trailing summary rows (lines beginning with a double underscore) before joining; for FPKM data, merge with dplyr's full_join so every gene present in any sample is retained, filling absent entries with NA or 0. Assemble the outer-joined table into a gene-by-sample matrix with the gene identifier as the first column and write it to merged_gene_counts.csv or merged_fpkm_matrix.csv. NASQAR exposes this as the GeneCountMerger Shiny app, which requires that the first column contains the genes and that gene columns match across all files.

## Related tools

- **GeneCountMerger** (NASQAR Shiny preprocessing app that merges individual HTSeq-style gene count files into one matrix) — https://github.com/nasqar/GeneCountMerger
- **NASQAR** (Web platform hosting the preprocessing app so non-programming experts can run it without coding) — https://github.com/nasqar/NASQAR
- **R** (Implementation language; dplyr-style full_join operations assemble the merged genes-by-samples matrix)
- **DESeq2Shiny** (Downstream bulk RNA-seq app that directly consumes the merged CSV count/FPKM matrix) — https://github.com/nasqar/deseq2shiny

## Examples

```
library(dplyr); library(readr); merged <- Reduce(function(x,y) full_join(x,y,by="gene_id"), lapply(count_files, read_tsv)); write_csv(merged, "merged_gene_counts.csv")
```

## Evaluation signals

- Output CSV is parseable and exists (merged_gene_counts.csv or merged_fpkm_matrix.csv)
- Row count of the merged matrix equals the number of unique gene identifiers across all input samples
- Matrix contains exactly one column per input sample, with the gene identifier as the first column
- All count entries are non-negative integers (for the count-merging path); HTSeq summary rows (double-underscore lines) are absent
- Genes absent from a given sample are consistently filled with NA or 0 rather than silently dropped

## Limitations

- GeneCountMerger requires the gene columns to match in all input files — otherwise the tool will not work
- The first column of every input file must contain the gene identifiers
- GeneCountMerger is only for bulk per-sample count files (e.g., HTSeq output), not single-cell or raw-read inputs

## Evidence

- [readme] This is a simple **preprocessing** tool to merge individual **gene count files** (Eg. output count files from htseq): [evidence span withheld — anchor status: external_doc]
- [readme] first column must contain the genes. If the gene columns do not match in all files, this tool will not work: [evidence span withheld — anchor status: external_doc]
- [methods] Functionality includes merging gene counts, conversion of gene IDs to gene names: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
- [intro] By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently: "By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently"
