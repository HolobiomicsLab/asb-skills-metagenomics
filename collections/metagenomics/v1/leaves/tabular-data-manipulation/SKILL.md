---
name: tabular-data-manipulation
description: Use when you have multiple per-sample HTSeq gene count files and need
  a single samples-by-gene count matrix (CSV) as input for downstream RNA-seq analysis
  such as DESeq2Shiny.
when_to_use_negative:
- The data is already a merged samples-by-gene count matrix — this skill only consolidates
  individual per-sample count files.
- Gene identifier columns differ across input files — per the README, 'If the gene
  columns do not match in all files, this tool will not work'.
- The goal is differential expression or visualization rather than preprocessing/merging
  — use DESeq2Shiny, SeuratV3Wizard, or other NASQAR apps instead.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: NASQAR
  role: Web platform hosting the GeneCountMerger preprocessing app that performs the
    count-file merge interactively
  repo: https://github.com/nasqar/NASQAR
- name: R
  role: Language used to load the two-column count tables, outer-join them on gene
    identifiers with dplyr-style manipulation, and write the merged matrix
- name: GeneCountMerger
  role: Preprocessing tool that merges individual gene count files (e.g., HTSeq output)
    into a single matrix
  repo: https://github.com/nasqar/GeneCountMerger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:451ec7f001670d4a78c16b217ae88b7ddf7c70a8c8c8ab634065ca0016e00f3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - GeneCountMerger
  - NASQAR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tabular-data-manipulation

## Summary

Merging per-sample HTSeq-style two-column gene count files (gene identifier, raw count) into a single gene-by-sample count matrix for downstream transcriptomics analysis, as implemented by NASQAR's GeneCountMerger preprocessing app.

## When to use

Use when you have multiple per-sample HTSeq gene count files and need a single samples-by-gene count matrix (CSV) as input for downstream RNA-seq analysis such as DESeq2Shiny.

## When NOT to use

- The data is already a merged samples-by-gene count matrix — this skill only consolidates individual per-sample count files.
- Gene identifier columns differ across input files — per the README, 'If the gene columns do not match in all files, this tool will not work'.
- The goal is differential expression or visualization rather than preprocessing/merging — use DESeq2Shiny, SeuratV3Wizard, or other NASQAR apps instead.

## Inputs

- Per-sample HTSeq-style gene count files (two-column: gene identifier, raw count)
- Example per-sample HTSeq count files from the GeneCountMerger use case

## Outputs

- merged_gene_counts.csv: a gene-by-sample integer count matrix (one row per unique gene, one column per input sample)

## How to apply

Load each per-sample HTSeq count file into R as a two-column table (gene identifier, raw count), dropping HTSeq trailing summary rows that begin with double underscores. Outer-join all sample tables on the shared first-column gene identifier key so that every gene present in any sample is retained. Assemble the joined table into a gene-by-sample integer count matrix using dplyr-style manipulation and write it to merged_gene_counts.csv. The NASQAR GeneCountMerger app provides this same merge as a web interface, and its README warns that the gene column must match across all files for the merge to succeed.

## Related tools

- **NASQAR** (Web platform hosting the GeneCountMerger preprocessing app that performs the count-file merge interactively) — https://github.com/nasqar/NASQAR
- **R** (Language used to load the two-column count tables, outer-join them on gene identifiers with dplyr-style manipulation, and write the merged matrix)
- **GeneCountMerger** (Preprocessing tool that merges individual gene count files (e.g., HTSeq output) into a single matrix) — https://github.com/nasqar/GeneCountMerger

## Evaluation signals

- Output matrix has exactly one row per unique gene identifier across all input files.
- Output matrix has exactly one column per input sample.
- All matrix entries are non-negative integer counts.
- Dropping HTSeq summary rows: no gene identifiers beginning with a double underscore appear in the output.
- Loading merged_gene_counts.csv yields a valid rectangular table suitable as input to downstream NASQAR apps such as DESeq2Shiny.

## Limitations

- Gene identifier columns must match across all input files; otherwise the merge fails.
- HTSeq trailing summary-statistics rows (double-underscore lines) must be excluded, not merged as genes.
- READMEs were truncated in context, so exact CLI/launch options for GeneCountMerger could not be confirmed.

## Evidence

- [methods] Functionality includes merging gene counts, conversion of gene IDs to gene names: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
- [readme] README tool description: [evidence span withheld — anchor status: external_doc]
- [readme] README constraint: [evidence span withheld — anchor status: external_doc]
- [readme] README tool framing: [evidence span withheld — anchor status: external_doc]
