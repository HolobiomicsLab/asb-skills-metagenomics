---
name: htseq-count-file-parsing
description: Use when the input consists of multiple per-sample HTSeq output count
  files that must be combined into one matrix of samples and corresponding gene counts
  before differential expression or enrichment analysis.
when_to_use_negative:
- The input is already a single gene-by-sample count matrix and only needs normalization,
  filtering, or differential expression analysis (use NASQAR's DESeq2Shiny or downstream
  apps instead)
- The individual count files have non-matching gene columns — the GeneCountMerger
  README states 'If the gene columns do not match in all files, this tool will not
  work'
- The task is conversion of gene IDs to gene names rather than merging count files
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GeneCountMerger
  role: NASQAR preprocessing app that merges individual HTSeq gene count files into
    a merged gene-by-sample count CSV matrix
  repo: https://github.com/nasqar/GeneCountMerger
- name: R
  role: Language in which the per-sample two-column count tables are loaded, parsed
    (dropping double-underscore HTSeq summary rows), outer-joined on gene identifier,
    and assembled into the count matrix
- name: NASQAR
  role: Web-based platform hosting GeneCountMerger as part of its Transcriptome Data
    Preprocessing toolbox
  repo: https://github.com/nasqar/NASQAR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:ffe6c122ca2648c6662907de47d9b54c3c8f5ae82170d837eac16e16669efe28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/GeneCountMerger
  tools:
  - GeneCountMerger
  - NASQAR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# htseq-count-file-parsing

## Summary

Parses per-sample HTSeq-style gene count files (two-column tables of gene identifier and raw count) and merges them into a single gene-by-sample count matrix, as implemented in NASQAR's GeneCountMerger preprocessing tool, to prepare input for downstream RNA-seq differential expression analysis.

## When to use

Use when the input consists of multiple per-sample HTSeq output count files that must be combined into one matrix of samples and corresponding gene counts before differential expression or enrichment analysis.

## When NOT to use

- The input is already a single gene-by-sample count matrix and only needs normalization, filtering, or differential expression analysis (use NASQAR's DESeq2Shiny or downstream apps instead)
- The individual count files have non-matching gene columns — the GeneCountMerger README states 'If the gene columns do not match in all files, this tool will not work'
- The task is conversion of gene IDs to gene names rather than merging count files

## Inputs

- Per-sample HTSeq-style gene count files (two-column text tables: gene identifier in the first column, raw integer count)
- Multiple input files whose gene columns match across all samples

## Outputs

- A merged gene-by-sample integer count matrix (e.g. merged_gene_counts.csv) with one row per unique gene identifier and one column per input sample

## How to apply

Load each HTSeq count file into R as a two-column table (gene identifier, raw count) and drop the trailing HTSeq summary rows that begin with double underscores (e.g. __no_feature, __ambiguous), since these are not genuine genes. Parse the first column as the gene key — the GeneCountMerger README requires that the first column contain the genes and that the gene columns match across all files. Outer-join all sample tables on the shared gene identifier so every gene present in any sample is retained as a row. Assemble the joined table into a gene-by-sample integer count matrix and write it out (GeneCountMerger produces a CSV such as merged_gene_counts.csv). The rationale is NASQAR's modular design: decoupling preprocessing from downstream apps lets the merged matrix feed DESeq2Shiny or other analysis tools independently.

## Related tools

- **GeneCountMerger** (NASQAR preprocessing app that merges individual HTSeq gene count files into a merged gene-by-sample count CSV matrix) — https://github.com/nasqar/GeneCountMerger
- **R** (Language in which the per-sample two-column count tables are loaded, parsed (dropping double-underscore HTSeq summary rows), outer-joined on gene identifier, and assembled into the count matrix)
- **NASQAR** (Web-based platform hosting GeneCountMerger as part of its Transcriptome Data Preprocessing toolbox) — https://github.com/nasqar/NASQAR

## Examples

```
sample <- read.table("sample1.counts", header=FALSE, col.names=c("gene","count")); sample <- sample[!grepl("^__", sample$gene), ]
```

## Evaluation signals

- Output matrix has exactly one row per unique gene identifier across all input files
- Output matrix has one column per input sample
- All matrix entries are non-negative integers (raw counts)
- HTSeq trailing summary rows (lines beginning with double underscore, e.g. __no_feature) are absent from the merged output
- Every gene appearing in any single sample file is present in at least one row of the merged matrix (outer-join completeness)

## Limitations

- Gene columns must match across all input files; per the README, 'If the gene columns do not match in all files, this tool will not work'
- HTSeq trailing summary-statistics lines (double-underscore prefixed) must be dropped or they will pollute the merged gene rows
- The tool is limited to merging; conversion of gene IDs to gene names is a separate NASQAR functionality

## Evidence

- [readme] README note on gene column requirement: [evidence span withheld — anchor status: not_found]
- [readme] README tool purpose: [evidence span withheld — anchor status: external_doc]
- [methods] Article methods on preprocessing functionality: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
- [intro] Article intro on modular design: "By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently"
