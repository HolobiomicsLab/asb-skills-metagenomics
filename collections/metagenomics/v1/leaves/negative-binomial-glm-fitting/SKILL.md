---
name: negative-binomial-glm-fitting
description: Use when raw gene-level RNA-seq count matrices plus sample metadata are
  available and per-gene log2 fold changes and Benjamini-Hochberg adjusted p-values
  are needed for differential expression between conditions.
when_to_use_negative:
- Input is single-cell RNA-seq data — NASQAR wraps scRNA analysis through SeuratV3Wizard/SeuratWizard
  rather than the DESeq2 workflow.
- Input is already a normalized/transformed expression table rather than raw counts,
  since the negative binomial model requires raw count data.
- Gene count columns do not match across files that need merging first — use GeneCountMerger
  preprocessing instead (first column must contain the genes).
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: NASQAR
  role: Web-based platform hosting the DESeq2Shiny app that wraps the negative binomial
    GLM fitting for bulk RNA-seq
  repo: https://github.com/nasqar/NASQAR
- name: deseq2shiny
  role: R Shiny wrapper around DESeq2 for RNA-seq counts exploratory analysis and
    differential expression
  repo: https://github.com/nasqar/deseq2shiny
- name: DEApp
  role: Shiny app conducting DE analysis with edgeR, limma-voom, and DESeq2 and cross-validating
    results across the three methods
  repo: https://github.com/yan-cri/DEApp
- name: GeneCountMerger
  role: Preprocessing tool to merge individual gene count files (e.g. htseq output)
    into a single count matrix for the DE step
  repo: https://github.com/nasqar/GeneCountMerger
- name: R
  role: Runtime environment for the DESeq2-backed Shiny applications
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:8214d14eb8b316aa259f295667b3f639013ca4e4a7f16ac7999d0e254d4d8b3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - DEApp
  - DESeq2
  - GeneCountMerger
  - NASQAR
  - R
  - deseq2shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# negative-binomial-glm-fitting

## Summary

Fits a negative binomial generalized linear model to bulk RNA-seq gene count data to estimate per-gene differential expression, as wrapped by NASQAR's DESeq2Shiny/DEApp applications around the DESeq2 R package ('Differential gene expression analysis based on the negative binomial distribution').

## When to use

Use when raw gene-level RNA-seq count matrices plus sample metadata are available and per-gene log2 fold changes and Benjamini-Hochberg adjusted p-values are needed for differential expression between conditions.

## When NOT to use

- Input is single-cell RNA-seq data — NASQAR wraps scRNA analysis through SeuratV3Wizard/SeuratWizard rather than the DESeq2 workflow.
- Input is already a normalized/transformed expression table rather than raw counts, since the negative binomial model requires raw count data.
- Gene count columns do not match across files that need merging first — use GeneCountMerger preprocessing instead (first column must contain the genes).

## Inputs

- csv matrix of samples and raw gene counts (first column containing genes)
- optional metadata csv with experimental group factor information for each sample
- design formula specifying the condition(s) to model

## Outputs

- gene-level DGE results table with gene identifier, base mean, log2FoldChange, p-value, and Benjamini-Hochberg adjusted p-value (de_results_table.csv)
- VST-transformed matrix
- Rlog-transformed matrix
- gene expression table
- normalized counts matrix

## How to apply

Import a csv count matrix of samples and gene counts (optionally with a metadata csv of experimental factors) into a DESeq2-backed workflow such as NASQAR's deseq2shiny or DEApp. Specify the design formula from the experimental factors, and let DESeq2 estimate size factors and dispersion under its negative binomial model before fitting the GLM per gene. Extract per-gene log2 fold changes and Benjamini-Hochberg adjusted p-values and write the full results table (gene identifier, base mean, log2FoldChange, p-value, padj) to de_results_table.csv. DEApp additionally supports cross-validating DE results across edgeR, limma-voom, and DESeq2 using the same two input files ('Raw Count Data' and 'Meta-data Table').

## Related tools

- **NASQAR** (Web-based platform hosting the DESeq2Shiny app that wraps the negative binomial GLM fitting for bulk RNA-seq) — https://github.com/nasqar/NASQAR
- **deseq2shiny** (R Shiny wrapper around DESeq2 for RNA-seq counts exploratory analysis and differential expression) — https://github.com/nasqar/deseq2shiny
- **DEApp** (Shiny app conducting DE analysis with edgeR, limma-voom, and DESeq2 and cross-validating results across the three methods) — https://github.com/yan-cri/DEApp
- **GeneCountMerger** (Preprocessing tool to merge individual gene count files (e.g. htseq output) into a single count matrix for the DE step) — https://github.com/nasqar/GeneCountMerger
- **R** (Runtime environment for the DESeq2-backed Shiny applications)

## Examples

```
Rscript -e "source('install/prep.R'); shiny::runApp()"  # launch DEApp (DESeq2/edgeR/limma-voom) on raw count + meta-data files
```

## Evaluation signals

- de_results_table.csv exists and contains one row per gene in the input count matrix
- Results table includes populated log2FoldChange and padj columns
- All Benjamini-Hochberg adjusted p-values fall within [0, 1]
- A size-factor-adjusted normalized counts matrix is produced alongside the results table
- In DEApp, DE results can be cross-validated against edgeR and limma-voom runs on the same two input files

## Limitations

- DESeq2Shiny is an exploratory/differential-expression wrapper; gene count files whose gene columns do not match cannot be merged directly by GeneCountMerger
- DEApp requires R version >= 3.2 and depends on shiny, shinydashboard, DT, ggplot2, edgeR, limma, and DESeq2 packages being installed
- SeuratWizard (the older single-cell counterpart) is no longer supported, so this skill is limited to bulk RNA-seq counts

## Evidence

- [readme] DESeq2Shiny: a web-based app based on the DESeq2 R package for RNA-seq counts data exploratory analysis and differential expression: [evidence span withheld — anchor status: external_doc]
- [readme] This Shiny app is a wrapper around DESeq2, an R package for differential gene expression analysis based on the negative binomial distribution: [evidence span withheld — anchor status: external_doc]
- [supplementary] Deseq2Shiny accepts a csv matrix of samples and gene counts plus an optional metadata csv, and outputs a VST matrix, Rlog matrix, DGE results table, gene expression table, and normalized counts matrix.: [evidence span withheld — anchor status: not_found]
- [readme] DEApp conducts differential expression analysis with edgeR, limma-voom, and DESeq2 and cross-validates results across the three methods: [evidence span withheld — anchor status: external_doc]
- [results] Users can upload their own DESeq2 output or import analyzed data from the upstream DESeq2Shiny app for enrichment analysis: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
