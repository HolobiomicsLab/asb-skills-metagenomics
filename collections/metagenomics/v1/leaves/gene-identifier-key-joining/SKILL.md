---
name: gene-identifier-key-joining
description: Use when you hold multiple per-sample two-column gene count files (gene
  identifier, raw count) that must be combined into one gene x sample matrix before
  downstream RNA-seq analysis.
when_to_use_negative:
- Gene columns (identifiers) do not match in all files — the README states 'this tool
  will not work' in that case
- Input is already a merged count matrix or feature-by-sample table rather than per-sample
  two-column count files
- The task requires normalization, filtering, or differential expression — use NASQAR's
  analysis apps (e.g., DESeq2Shiny) instead; NASQAR explicitly decouples preprocessing
  from analysis
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NASQAR
  role: Web-based platform hosting GeneCountMerger as a preprocessing application
    so non-programming experts can merge gene counts without coding
  repo: https://github.com/nasqar/NASQAR
- name: R
  role: Language used to parse the two-column count tables, outer-join them on the
    gene identifier key, and assemble the merged matrix
  repo: https://github.com/nasqar/GeneCountMerger
- name: GeneCountMerger
  role: Standalone preprocessing tool that merges individual gene count files (e.g.,
    HTSeq output) into a single count matrix
  repo: https://github.com/nasqar/GeneCountMerger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:a5ba23d36db22d834efbde5b105356a5a41b6b05fa41fd26d6ab1e6c43faa41e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - GeneCountMerger
  - NASQAR
  - R
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-identifier-key-joining

## Summary

Merge per-sample gene count files (e.g., HTSeq output) into a single gene-by-sample count matrix by outer-joining two-column tables on the shared gene identifier key, as implemented in NASQAR's GeneCountMerger preprocessing tool.

## When to use

Use when you hold multiple per-sample two-column gene count files (gene identifier, raw count) that must be combined into one gene x sample matrix before downstream RNA-seq analysis.

## When NOT to use

- Gene columns (identifiers) do not match in all files — the README states 'this tool will not work' in that case
- Input is already a merged count matrix or feature-by-sample table rather than per-sample two-column count files
- The task requires normalization, filtering, or differential expression — use NASQAR's analysis apps (e.g., DESeq2Shiny) instead; NASQAR explicitly decouples preprocessing from analysis

## Inputs

- Individual per-sample gene count files (e.g., HTSeq output count files), each with the gene identifiers in the first column and raw counts in the second
- Consistent gene identifier sets across all input files

## Outputs

- A merged gene-by-sample integer count matrix written to merged_gene_counts.csv, with one row per unique gene identifier across all inputs and one column per input sample

## How to apply

Load each per-sample HTSeq-style count file as a two-column table (gene identifier, raw count) and drop HTSeq trailing summary rows (lines beginning with a double underscore). Outer-join all sample tables on the gene identifier key so every gene appearing in any sample is retained as a row — GeneCountMerger requires the first column to contain the genes and warns the tool will not work if the gene columns do not match across files. Assemble the joined table into a gene-by-sample integer count matrix using dplyr-style manipulation in R, then write it out as merged_gene_counts.csv. NASQAR wraps this as a web-based preprocessing step so counts are ready for tools like DESeq2Shiny.

## Related tools

- **NASQAR** (Web-based platform hosting GeneCountMerger as a preprocessing application so non-programming experts can merge gene counts without coding) — https://github.com/nasqar/NASQAR
- **R** (Language used to parse the two-column count tables, outer-join them on the gene identifier key, and assemble the merged matrix) — https://github.com/nasqar/GeneCountMerger
- **GeneCountMerger** (Standalone preprocessing tool that merges individual gene count files (e.g., HTSeq output) into a single count matrix) — https://github.com/nasqar/GeneCountMerger

## Examples

```
merged <- Reduce(function(x, y) merge(x, y, by = "gene", all = TRUE), list(s1, s2)); write.csv(merged, "merged_gene_counts.csv", row.names = FALSE)
```

## Evaluation signals

- Output matrix has exactly one row per unique gene identifier across all input files (no duplicated keys after the outer join)
- Output matrix has exactly one column per input sample
- All entries in the matrix are non-negative integer counts
- HTSeq trailing summary-statistics rows (lines beginning with '__', e.g., __no_feature, __ambiguous) are absent from the merged output
- Genes present in only a subset of samples appear as rows with missing/NA or equivalent placeholders in the other sample columns

## Limitations

- The first column of every input file must contain the genes; if the gene columns do not match in all files, the tool will not work (README NOTE)
- The tool only merges counts — it does not perform filtering, normalization, or gene ID conversion, which are separate NASQAR functionalities
- No concrete command-line or installation parameters are given in the truncated README; usage is via the online demo or the R source

## Evidence

- [readme] README note on gene column requirement: [evidence span withheld — anchor status: external_doc]
- [readme] README tool scope: [evidence span withheld — anchor status: not_found]
- [methods] Article methods on merging functionality: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
- [intro] Article intro on modular decoupling of preprocessing from analysis: "By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently"
