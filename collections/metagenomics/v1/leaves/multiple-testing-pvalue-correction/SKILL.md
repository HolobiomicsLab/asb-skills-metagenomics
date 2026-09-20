---
name: multiple-testing-pvalue-correction
description: Use when a gene-level differential expression results table contains
  raw p-values from a negative-binomial or similar GLM fit and per-gene FDR-adjusted
  values (padj) are needed for downstream filtering or enrichment input.
when_to_use_negative:
- Input is already a table of adjusted p-values (padj) exported from DESeq2Shiny —
  no second correction should be applied
- Analysis goal is single-cell RNA-seq (use the Seurat-based SeuratV3Wizard path in
  NASQAR instead of DESeq2Shiny)
- Input is pre-processed metagenomic count data, where SHAMAN performs its own differential
  analysis
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: NASQAR
  role: Web-based platform hosting the Deseq2Shiny differential-expression app that
    performs the BH correction
  repo: https://github.com/nasqar/NASQAR
- name: DESeq2
  role: R package performing negative-binomial differential gene expression analysis
    whose results carry the Benjamini-Hochberg adjusted p-values
  repo: https://github.com/nasqar/deseq2shiny
- name: Deseq2Shiny
  role: Shiny wrapper around DESeq2 that accepts a count matrix plus metadata csv
    and outputs the DGE results table with padj
  repo: https://github.com/nasqar/deseq2shiny
- name: DEApp
  role: Alternative Shiny app running DESeq2 (and edgeR/limma-voom) differential expression
    with cross-validation across the three methods
  repo: https://github.com/yan-cri/DEApp
- name: ClusterProfShinyORA
  role: Downstream enrichment app that can import the DESeq2/Deseq2Shiny output for
    over-representation analysis
  repo: https://github.com/nasqar/ClusterProfShinyORA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:9c503b6b687b492bae0e07294552a40b08d5e95cc3abfd8bc3f5f14ddcd2cdd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - ClusterProfShinyORA
  - DEApp
  - DESeq2
  - Deseq2Shiny
  - NASQAR
  - R
  - deseq2shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multiple-testing-pvalue-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate adjustment to per-gene p-values produced by bulk RNA-seq differential expression analysis (e.g., the DESeq2 negative-binomial model wrapped by NASQAR's Deseq2Shiny/DEApp), yielding per-gene adjusted p-values that control for thousands of simultaneous gene-level tests.

## When to use

Use when a gene-level differential expression results table contains raw p-values from a negative-binomial or similar GLM fit and per-gene FDR-adjusted values (padj) are needed for downstream filtering or enrichment input.

## When NOT to use

- Input is already a table of adjusted p-values (padj) exported from DESeq2Shiny — no second correction should be applied
- Analysis goal is single-cell RNA-seq (use the Seurat-based SeuratV3Wizard path in NASQAR instead of DESeq2Shiny)
- Input is pre-processed metagenomic count data, where SHAMAN performs its own differential analysis

## Inputs

- csv count matrix of samples and gene counts (bulk RNA-seq)
- optional sample metadata csv with experimental group factor information
- per-gene DE results table with columns including log2FoldChange, p-value, and padj

## Outputs

- de_results_table.csv with one row per gene containing gene identifier, base mean, log2 fold change, p-value, and Benjamini-Hochberg adjusted p-value
- DGE results table exported from Deseq2Shiny

## How to apply

Load the raw count matrix and sample metadata into a DESeq2-backed interface (NASQAR Deseq2Shiny or DEApp), specify the design formula, and let the workflow estimate size factors and dispersion under the negative-binomial model. Fit the differential-expression model to obtain per-gene log2 fold changes and raw p-values, then apply the Benjamini-Hochberg procedure over all tested genes to produce adjusted p-values. Export the full gene-level results table (gene identifier, base mean, log2 fold change, p-value, adjusted p-value) as de_results_table.csv. The rationale is that gene-level tests are simultaneous across the whole transcriptome, so uncorrected p-values are unusable; the BH-adjusted values are what downstream tools (e.g., ClusterProfilerShiny) expect, and the article notes users can upload DESeq2 output directly into enrichment analysis.

## Related tools

- **NASQAR** (Web-based platform hosting the Deseq2Shiny differential-expression app that performs the BH correction) — https://github.com/nasqar/NASQAR
- **DESeq2** (R package performing negative-binomial differential gene expression analysis whose results carry the Benjamini-Hochberg adjusted p-values) — https://github.com/nasqar/deseq2shiny
- **Deseq2Shiny** (Shiny wrapper around DESeq2 that accepts a count matrix plus metadata csv and outputs the DGE results table with padj) — https://github.com/nasqar/deseq2shiny
- **DEApp** (Alternative Shiny app running DESeq2 (and edgeR/limma-voom) differential expression with cross-validation across the three methods) — https://github.com/yan-cri/DEApp
- **ClusterProfShinyORA** (Downstream enrichment app that can import the DESeq2/Deseq2Shiny output for over-representation analysis) — https://github.com/nasqar/ClusterProfShinyORA

## Examples

```
Rscript -e 'shiny::runApp("~/DEAPP-shiny-master")'  # upload raw count + meta-data table via the Data Input tab; export DGE table as de_results_table.csv
```

## Evaluation signals

- de_results_table.csv exists and contains one row per gene present in the input count matrix
- The log2FoldChange and padj columns are populated (non-null) for all genes
- All adjusted p-values fall within the interval [0, 1]
- The results table includes the expected columns: gene identifier, base mean, log2 fold change, p-value, adjusted p-value
- padj values are >= the corresponding raw p-values for each gene (monotonicity of BH adjustment)

## Limitations

- GeneCountMerger-style preprocessing requires the first column to contain the genes, and if gene columns do not match in all files the upstream step will not work
- DEApp requires R version >= 3.2 and the CRAN/Bioconductor packages shiny, shinydashboard, DT, ggplot2, edgeR, limma, DESeq2
- DESeq2Shiny is an exploratory, GUI-level wrapper; results depend on the design formula supplied by the user in the supplementary example use case

## Evidence

- [supplementary] Fit the differential-expression model and extract per-gene log2 fold changes and Benjamini-Hochberg adjusted p-values.: [evidence span withheld — anchor status: not_found]
- [supplementary] Deseq2Shiny outputs DGE results table: [evidence span withheld — anchor status: not_found]
- [readme] DESeq2 negative binomial description: [evidence span withheld — anchor status: external_doc]
- [results] ClusterProfilerShiny imports DESeq2Shiny output: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
- [supplementary] adjusted p-value range validation: [evidence span withheld — anchor status: not_found]
