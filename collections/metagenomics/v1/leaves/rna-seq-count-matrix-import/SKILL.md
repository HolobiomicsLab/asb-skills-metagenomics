---
name: rna-seq-count-matrix-import
description: Use when the input is a raw/normalized RNA-seq count matrix (csv/txt,
  genes in rows or first column, one column per sample) and the goal is to run differential
  expression via DESeq2, optionally with a metadata table of experimental factors.
when_to_use_negative:
- The data are single-cell (e.g. a .h5ad or 10x matrix) — use NASQAR's SeuratV3Wizard
  for scRNA-seq instead of the bulk DESeq2Shiny app
- The input is already DESeq2 output; for downstream analysis, upload it to ClusterProfilerShiny
  rather than re-importing as counts
- Individual count files have non-matching gene columns — merge them first with GeneCountMerger,
  which will not work on mismatched gene columns
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NASQAR
  role: web-based platform hosting the DESeq2Shiny bulk RNA-seq differential-expression
    app used for the count-matrix import
  repo: https://github.com/nasqar/NASQAR
- name: DESeq2Shiny
  role: R Shiny wrapper around DESeq2 that accepts the uploaded count matrix plus
    optional metadata and performs exploratory analysis and differential expression
  repo: https://github.com/nasqar/deseq2shiny
- name: DEApp
  role: alternative Shiny app for count-matrix import supporting DE analysis with
    edgeR, limma-voom, and DESeq2 and cross-validation of results across the three
    methods
  repo: https://github.com/yan-cri/DEApp
- name: GeneCountMerger
  role: preprocessing tool to merge individual gene count files (e.g. htseq output)
    into a single count matrix before import; requires matching gene columns
  repo: https://github.com/nasqar/GeneCountMerger
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:a32bbc5f8b409bbe491961451f8fe9840954951574c8b4e4fd3db8be902e0063
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - DEApp
  - DESeq2
  - DESeq2Shiny
  - GeneCountMerger
  - NASQAR
  - R
  - deseq2shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# rna-seq-count-matrix-import

## Summary

Import a bulk RNA-seq gene count matrix (with optional sample metadata) into a DESeq2-backed web application (NASQAR's DESeq2Shiny/DEApp) so that exploratory analysis and differential expression can be run through a GUI without programming.

## When to use

Use when the input is a raw/normalized RNA-seq count matrix (csv/txt, genes in rows or first column, one column per sample) and the goal is to run differential expression via DESeq2, optionally with a metadata table of experimental factors.

## When NOT to use

- The data are single-cell (e.g. a .h5ad or 10x matrix) — use NASQAR's SeuratV3Wizard for scRNA-seq instead of the bulk DESeq2Shiny app
- The input is already DESeq2 output; for downstream analysis, upload it to ClusterProfilerShiny rather than re-importing as counts
- Individual count files have non-matching gene columns — merge them first with GeneCountMerger, which will not work on mismatched gene columns

## Inputs

- csv or txt gene count matrix (samples x genes, genes in the first column)
- optional sample metadata table (csv/txt) with experimental group factor per sample
- design formula / experimental factor specification for the DE model

## Outputs

- VST-transformed count matrix
- Rlog-transformed count matrix
- differential-expression results table (gene identifier, base mean, log2FoldChange, p-value, padj)
- gene expression table
- normalized counts matrix

## How to apply

Prepare a csv matrix of samples and gene counts where the first column contains the gene identifiers; if the count columns come from separate htseq-style files, first merge them with GeneCountMerger, which requires gene columns to match across files. Optionally supply a metadata csv/txt giving the experimental group factor for each sample, which DEApp and DESeq2Shiny use to define the design. Upload both files via the app's Data Input tab (DEApp) or the corresponding upload step in DESeq2Shiny, and specify the design formula documented for the use case. DESeq2 then estimates size factors and dispersion under its negative binomial model and fits the differential-expression model, yielding per-gene log2 fold changes and Benjamini-Hochberg adjusted p-values.

## Related tools

- **NASQAR** (web-based platform hosting the DESeq2Shiny bulk RNA-seq differential-expression app used for the count-matrix import) — https://github.com/nasqar/NASQAR
- **DESeq2Shiny** (R Shiny wrapper around DESeq2 that accepts the uploaded count matrix plus optional metadata and performs exploratory analysis and differential expression) — https://github.com/nasqar/deseq2shiny
- **DEApp** (alternative Shiny app for count-matrix import supporting DE analysis with edgeR, limma-voom, and DESeq2 and cross-validation of results across the three methods) — https://github.com/yan-cri/DEApp
- **GeneCountMerger** (preprocessing tool to merge individual gene count files (e.g. htseq output) into a single count matrix before import; requires matching gene columns) — https://github.com/nasqar/GeneCountMerger

## Examples

```
Rscript -e "shiny::runApp('~/Desktop/DEAPP-shiny-master')"  # then upload Raw Count Data + Meta-data Table via the Data Input tab
```

## Evaluation signals

- The app accepts the uploaded matrix without errors, confirming the first column contains gene identifiers and count columns align with sample names
- The output de_results_table contains one row per gene in the input count matrix with log2FoldChange and padj columns populated
- All Benjamini-Hochberg adjusted p-values fall within [0, 1]
- VST and Rlog matrices are produced with the same dimensions as the input count matrix
- If DEApp is used with a single-factor design, results are consistent with its negative binomial (DESeq2) expectations and cross-validation against edgeR/limma-voom runs

## Limitations

- GeneCountMerger requires the first column to contain genes and will not work if the gene columns do not match across count files
- DEApp requires R >= 3.2 and installation of CRAN (shiny, shinydashboard, DT, ggplot2) and Bioconductor (edgeR, limma, DESeq2) packages before launch
- The supplementary example use case (Additional file 1) documents the design, but the NASQAR article does not state software versions for deployment technologies (Docker/Swarm, Kubernetes)

## Evidence

- [other] Deseq2Shiny accepts a csv matrix of samples and gene counts plus an optional metadata csv: [evidence span withheld — anchor status: not_found]
- [readme] DESeq2Shiny app description: [evidence span withheld — anchor status: external_doc]
- [readme] DEApp input files: [evidence span withheld — anchor status: external_doc]
- [readme] GeneCountMerger constraint: [evidence span withheld — anchor status: external_doc]
- [results] downstream enrichment integration: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
