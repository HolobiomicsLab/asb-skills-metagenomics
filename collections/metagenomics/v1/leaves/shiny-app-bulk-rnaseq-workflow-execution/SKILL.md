---
name: shiny-app-bulk-rnaseq-workflow-execution
description: Use when the task requires running a bulk RNA-seq differential-expression
  analysis on a gene count matrix with sample metadata through a GUI-driven Shiny
  app rather than hand-written R scripts.
when_to_use_negative:
- Input is single-cell RNA-seq data — use NASQAR's SeuratV3Wizard instead, which is
  the single-cell Shiny interface to Seurat.
- The task is count preprocessing/merging rather than differential expression — GeneCountMerger
  handles merging individual gene count files (e.g., htseq output), and it will not
  work if the gene columns do not match across files.
- A scripted, reproducible DESeq2 pipeline with custom model fitting is required —
  the Shiny wrapper is aimed at researchers avoiding code, not at programmatic workflow
  execution.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: NASQAR
  role: web-based platform hosting the DESeq2Shiny app for interactive bulk RNA-seq
    differential expression analysis
  repo: https://github.com/nasqar/NASQAR
- name: deseq2shiny
  role: Shiny app wrapping the DESeq2 R package for RNA-seq counts exploratory analysis
    and differential expression; the primary execution vehicle for this skill
  repo: https://github.com/nasqar/deseq2shiny
- name: DEApp
  role: alternative interactive Shiny app that performs DE analysis with edgeR, limma-voom,
    and DESeq2 and cross-validates results across the three methods
  repo: https://github.com/yan-cri/DEApp
- name: R
  role: runtime environment in which the Shiny apps and DESeq2 run (shiny::runApp());
    DEApp requires R version >= 3.2
- name: DESeq2
  role: underlying Bioconductor package performing differential gene expression analysis
    based on the negative binomial distribution
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:a6ca07586936b4620c80d124db3cab84b5d2edead6e8410091766fb7767186a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - DEApp
  - DESeq2
  - NASQAR
  - R
  - deseq2shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# shiny-app-bulk-rnaseq-workflow-execution

## Summary

Executes a bulk RNA-seq differential gene expression workflow through NASQAR's DESeq2Shiny Shiny app, which wraps the DESeq2 R package (negative binomial model) behind an intuitive GUI so non-programming experts can upload counts, run DE analysis, and export results without scripting.

## When to use

Use when the task requires running a bulk RNA-seq differential-expression analysis on a gene count matrix with sample metadata through a GUI-driven Shiny app rather than hand-written R scripts.

## When NOT to use

- Input is single-cell RNA-seq data — use NASQAR's SeuratV3Wizard instead, which is the single-cell Shiny interface to Seurat.
- The task is count preprocessing/merging rather than differential expression — GeneCountMerger handles merging individual gene count files (e.g., htseq output), and it will not work if the gene columns do not match across files.
- A scripted, reproducible DESeq2 pipeline with custom model fitting is required — the Shiny wrapper is aimed at researchers avoiding code, not at programmatic workflow execution.

## Inputs

- csv matrix of samples and gene counts (raw RNA-seq counts, first column contains genes)
- optional sample metadata csv with experimental group factor information
- design formula from the supplementary example use case

## Outputs

- gene-level differential expression results table (gene identifier, base mean, log2 fold change, p-value, adjusted p-value) written to de_results_table.csv
- VST-transformed matrix
- Rlog-transformed matrix
- gene expression table
- normalized counts matrix

## How to apply

Load the example count matrix and sample metadata from Supplementary Additional file 1 and launch the DESeq2-backed application (deseq2shiny, or DEApp which offers edgeR/limma-voom/DESeq2 cross-validation). Deseq2Shiny accepts a csv matrix of samples and gene counts plus an optional metadata csv; supply the design formula documented in the supplementary use case so DESeq2 estimates size factors and dispersion under its negative binomial model and fits the differential-expression model. Extract per-gene log2 fold changes and Benjamini-Hochberg adjusted p-values, then write the full gene-level results table (gene identifier, base mean, log2 fold change, p-value, adjusted p-value) to de_results_table.csv. The app also produces a VST matrix, Rlog matrix, gene expression table, and normalized counts matrix, which downstream NASQAR tools such as ClusterProfilerShiny can import directly.

## Related tools

- **NASQAR** (web-based platform hosting the DESeq2Shiny app for interactive bulk RNA-seq differential expression analysis) — https://github.com/nasqar/NASQAR
- **deseq2shiny** (Shiny app wrapping the DESeq2 R package for RNA-seq counts exploratory analysis and differential expression; the primary execution vehicle for this skill) — https://github.com/nasqar/deseq2shiny
- **DEApp** (alternative interactive Shiny app that performs DE analysis with edgeR, limma-voom, and DESeq2 and cross-validates results across the three methods) — https://github.com/yan-cri/DEApp
- **R** (runtime environment in which the Shiny apps and DESeq2 run (shiny::runApp()); DEApp requires R version >= 3.2)
- **DESeq2** (underlying Bioconductor package performing differential gene expression analysis based on the negative binomial distribution)

## Examples

```
Rscript -e "shiny::runApp('DEAPP-shiny-master')"
```

## Evaluation signals

- de_results_table.csv exists and contains one row per gene present in the input count matrix
- the results table includes populated log2FoldChange and padj columns
- all Benjamini-Hochberg adjusted p-values fall within [0, 1]
- companion outputs (VST matrix, Rlog matrix, normalized counts matrix) were produced by the app
- if using DEApp, cross-validation of DE results across edgeR, limma-voom, and DESeq2 can be inspected for consistency

## Limitations

- Deseq2Shiny is a wrapper around DESeq2 only; it does not offer multi-method comparison — that requires DEApp (edgeR, limma-voom, DESeq2).
- GeneCountMerger preprocessing upstream fails if gene columns do not match across input count files.
- The Supplementary Additional file 1 with example use cases is referenced but its section content was not extractable from the article, so the exact example count matrix and design formula must be obtained from the SI itself.

## Evidence

- [supplementary] Deseq2Shiny accepts a csv matrix of samples and gene counts plus an optional metadata csv: [evidence span withheld — anchor status: not_found]
- [results] Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app.: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
- [readme] conduct differential expression (DE) analysis with edgeR, limma-voom, and DESeq2: [evidence span withheld — anchor status: not_found]
- [discussion] Additional file 1 includes supplementary materials and example use cases for NASQAR: "Additional file 1 This file includes supplementary materials such as instructions on how to launch NASQAR and example use cases on data analysis and visualization."
- [abstract] NASQAR empowers non-programming experts to explore, analyze, and visualize transcriptomics data: "NASQAR empowers non-programming experts with a versatile and intuitive toolbox to easily and efficiently explore, analyze, and visualize their Transcriptomics data interactively"
