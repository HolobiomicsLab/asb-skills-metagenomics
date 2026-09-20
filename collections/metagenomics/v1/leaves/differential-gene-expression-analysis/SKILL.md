---
name: differential-gene-expression-analysis
description: Use when you have a raw bulk RNA-seq count matrix (csv/txt) plus optional
  sample metadata and need per-gene differential expression results (log2FoldChange,
  Benjamini-Hochberg adjusted p-values) without scripting R/DESeq2 directly.
when_to_use_negative:
- Input is single-cell RNA-seq data — use SeuratV3Wizard/SeuratWizard instead of DESeq2Shiny
- You only have individual per-sample gene count files (e.g., htseq output) that first
  need merging — use GeneCountMerger preprocessing first
- The goal is metagenomic differential analysis rather than gene-level RNA-seq — SHAMAN
  is the NASQAR tool for that
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: deseq2shiny
  role: NASQAR web app wrapping DESeq2 for bulk RNA-seq counts exploratory analysis
    and differential expression; accepts count matrix plus optional metadata and outputs
    VST/Rlog matrices, DGE results, gene expression table, and normalized counts
  repo: https://github.com/nasqar/deseq2shiny
- name: DEApp
  role: Interactive R Shiny app for differential expression analysis with edgeR, limma-voom,
    and DESeq2 and cross-validation of results across the three methods
  repo: https://github.com/yan-cri/DEApp
- name: GeneCountMerger
  role: Preprocessing tool to merge individual gene count files (e.g., htseq output)
    into a single count matrix usable as DESeq2Shiny input
  repo: https://github.com/nasqar/GeneCountMerger
- name: ClusterProfShinyGSEA
  role: Downstream enrichment analysis that can import analyzed data directly from
    the upstream DESeq2Shiny app
  repo: https://github.com/nasqar/ClusterProfShinyGSEA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:7c451b09e4d36fbbae240034c118556d7f7ce274bc118e2e647c7c3e259be8ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/deseq2shiny
  tools:
  - ClusterProfShinyGSEA
  - DEApp
  - DESeq2
  - GeneCountMerger
  - NASQAR
  - R
  - deseq2shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# differential-gene-expression-analysis

## Summary

Perform bulk RNA-seq differential gene expression analysis using DESeq2-backed Shiny applications (deseq2shiny/DEApp) from the NASQAR platform, which wrap the negative-binomial DESeq2 model behind a GUI so researchers can upload count matrices and obtain per-gene log2 fold changes and adjusted p-values without programming.

## When to use

Use when you have a raw bulk RNA-seq count matrix (csv/txt) plus optional sample metadata and need per-gene differential expression results (log2FoldChange, Benjamini-Hochberg adjusted p-values) without scripting R/DESeq2 directly.

## When NOT to use

- Input is single-cell RNA-seq data — use SeuratV3Wizard/SeuratWizard instead of DESeq2Shiny
- You only have individual per-sample gene count files (e.g., htseq output) that first need merging — use GeneCountMerger preprocessing first
- The goal is metagenomic differential analysis rather than gene-level RNA-seq — SHAMAN is the NASQAR tool for that

## Inputs

- Raw count data file (.txt or .csv) with counts of all tags/genes per sample, genes in the first column
- Optional metadata table (.txt or .csv) with experimental group/factor information for each sample
- Design formula documenting the experimental factors (from the supplementary use case)

## Outputs

- Differential-expression results table with per-gene base mean, log2 fold change, p-value, and Benjamini-Hochberg adjusted p-value (e.g., de_results_table.csv)
- VST-transformed matrix
- Rlog-transformed matrix
- Gene expression table
- Normalized counts matrix

## How to apply

Import a csv/txt matrix of samples and gene counts (first column must contain the genes) into deseq2shiny or DEApp, optionally supplying a metadata csv with the experimental factor information. The app wraps DESeq2, which models counts with the negative binomial distribution: provide the design formula from your experimental design, then the app estimates size factors and dispersion and fits the differential-expression model. DEApp additionally allows cross-validating results across three methods (edgeR, limma-voom, DESeq2) on the same input files. Extract the full gene-level results table (gene identifier, base mean, log2 fold change, p-value, adjusted p-value) and write it out (e.g., de_results_table.csv). ClusterProfilerShiny can then import the analyzed data directly from the upstream DESeq2Shiny app for gene-enrichment follow-up.

## Related tools

- **deseq2shiny** (NASQAR web app wrapping DESeq2 for bulk RNA-seq counts exploratory analysis and differential expression; accepts count matrix plus optional metadata and outputs VST/Rlog matrices, DGE results, gene expression table, and normalized counts) — https://github.com/nasqar/deseq2shiny
- **DEApp** (Interactive R Shiny app for differential expression analysis with edgeR, limma-voom, and DESeq2 and cross-validation of results across the three methods) — https://github.com/yan-cri/DEApp
- **GeneCountMerger** (Preprocessing tool to merge individual gene count files (e.g., htseq output) into a single count matrix usable as DESeq2Shiny input) — https://github.com/nasqar/GeneCountMerger
- **ClusterProfShinyGSEA** (Downstream enrichment analysis that can import analyzed data directly from the upstream DESeq2Shiny app) — https://github.com/nasqar/ClusterProfShinyGSEA

## Examples

```
Rscript -e "setwd('~/Desktop/DEAPP-shiny-master'); shiny::runApp()"
```

## Evaluation signals

- de_results_table.csv exists and contains one row per gene in the input count matrix
- The results table includes the columns log2FoldChange and padj, both populated
- All adjusted p-values fall within [0, 1]
- Cross-checking DESeq2 results against edgeR and limma-voom in DEApp should show consistent direction of effect for robust hits (the cross-validation purpose stated in the DEApp README)
- Sample metadata factors match the samples/columns of the count matrix (GeneCountMerger requires matching gene columns across files; DEApp expects the metadata to carry group factor info for each sample)

## Limitations

- Deseq2Shiny/DEApp accept only raw count matrices (csv/txt); count files whose gene columns do not match cannot be merged (GeneCountMerger note: 'If the gene columns do not match in all files, this tool will not work')
- DEApp requires R version >= 3.2 and installation of CRAN packages (shiny, shinydashboard, DT, ggplot2) plus Bioconductor packages (edgeR, limma, DESeq2)
- The supplementary Additional file 1 with the example use cases and count matrices is referenced but its content is not fully present in the article text

## Evidence

- [supplementary] Deseq2Shiny (bulk RNA-seq DGE analysis) accepts a csv matrix of samples and gene counts plus an optional metadata csv, and outputs a VST matrix, Rlog matrix, DGE results table, gene expression table, and normalized counts matrix.: [evidence span withheld — anchor status: not_found]
- [readme] This Shiny app is a wrapper around DESeq2, an R package for "Differential gene expression analysis based on the negative binomial distribution".: [evidence span withheld — anchor status: external_doc]
- [readme] This DE analysis interactive web application (App) is developed in R with Shiny, aiming to 1). conduct differential expression (DE) analysis with edgeR, limma-voom, and DESeq2, and 2). cross-validate the DE analysis results: [evidence span withheld — anchor status: external_doc]
- [readme] The input of this App is 2 files in '.txt' or '.csv' format. 1. 'Raw Count Data' : includes the count results of all tags with respect to each sample. 2. 'Meta-data Table' : includes the experimental group factor information for each sample.: "The input of this App is 2 files in '.txt' or '.csv' format.

1. 'Raw Count Data' : includes the count results of all tags with respect to each sample.  
2. 'Meta-data Table' : includes the"
- [results] Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app.: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
