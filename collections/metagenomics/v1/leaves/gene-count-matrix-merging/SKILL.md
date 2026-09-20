---
name: gene-count-matrix-merging
description: Use when you hold one two-column gene abundance file per sample (gene
  identifier + raw count, or gene identifier + FPKM) and need a single merged matrix
  to feed differential expression tools like DESeq2Shiny.
when_to_use_negative:
- Input is already a single merged gene-by-sample matrix — no merging is needed
- Gene identifier columns do not match in all files — the README warns the tool will
  not work in this case
- Input is raw reads, single-cell count data, or already-normalized expression values
  rather than per-sample gene count/FPKM tables
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NASQAR GeneCountMerger
  role: Preprocessing app that merges individual gene count files (e.g. HTSeq output)
    into a sample-by-gene count matrix
  repo: https://github.com/nasqar/GeneCountMerger
- name: R
  role: Language used to load, outer-join (dplyr full_join on gene identifier), and
    write the merged matrix
- name: DESeq2Shiny
  role: Downstream bulk RNA-seq app that consumes the merged count matrix for exploratory
    analysis and differential expression
  repo: https://github.com/nasqar/deseq2shiny
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:1660a4433db145b9877e1353789e91b572fd5bade134227877e5d6073f5ef36f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/GeneCountMerger
  tools:
  - DESeq2Shiny
  - NASQAR
  - NASQAR GeneCountMerger
  - R
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gene-count-matrix-merging

## Summary

Merges per-sample gene count files (e.g. HTSeq output count files) or per-sample FPKM files into a single gene-by-sample matrix, as implemented by NASQAR's GeneCountMerger preprocessing app, so downstream bulk RNA-seq apps like DESeq2Shiny can consume the data.

## When to use

Use when you hold one two-column gene abundance file per sample (gene identifier + raw count, or gene identifier + FPKM) and need a single merged matrix to feed differential expression tools like DESeq2Shiny.

## When NOT to use

- Input is already a single merged gene-by-sample matrix — no merging is needed
- Gene identifier columns do not match in all files — the README warns the tool will not work in this case
- Input is raw reads, single-cell count data, or already-normalized expression values rather than per-sample gene count/FPKM tables

## Inputs

- Per-sample gene count files (e.g. HTSeq output count files), two-column tabular: gene identifier and raw count
- Per-sample gene FPKM files (e.g. Cufflinks-style tabular gene abundance output)
- Gene identifier column must be the first column and consistent across all files

## Outputs

- merged_gene_counts.csv — gene-by-sample integer count matrix with gene identifiers as the first column
- merged_fpkm_matrix.csv — gene-by-sample FPKM matrix (for FPKM inputs), consumable by downstream NASQAR apps such as DESeq2Shiny

## How to apply

Load each per-sample file into R as a two-column table keyed on the gene identifier column (the first column must contain the genes). For HTSeq-style counts, drop the trailing summary-statistics rows whose lines begin with a double underscore, keeping only genuine gene rows. Outer-join all sample tables on the shared gene identifier key (e.g. dplyr's full_join) so every gene appearing in any sample is retained, filling genes absent from a given sample with NA or 0. For FPKM inputs, do the same full outer join on the gene identifier, optionally converting gene IDs to gene names as GeneCountMerger/mergeFPKMs offer. Write the merged matrix to a CSV (e.g. merged_gene_counts.csv or merged_fpkm_matrix.csv) with the gene identifier as the first column and one column per sample, ready for NASQAR's downstream bulk RNA-seq apps.

## Related tools

- **NASQAR GeneCountMerger** (Preprocessing app that merges individual gene count files (e.g. HTSeq output) into a sample-by-gene count matrix) — https://github.com/nasqar/GeneCountMerger
- **R** (Language used to load, outer-join (dplyr full_join on gene identifier), and write the merged matrix)
- **DESeq2Shiny** (Downstream bulk RNA-seq app that consumes the merged count matrix for exploratory analysis and differential expression) — https://github.com/nasqar/deseq2shiny

## Examples

```
merged <- Reduce(function(a,b) dplyr::full_join(a,b,by="gene"), samples); write.csv(merged, "merged_gene_counts.csv", row.names=FALSE)
```

## Evaluation signals

- Output matrix has one row per unique gene identifier across all input samples
- Output matrix has one column per input sample plus the gene identifier as the first column
- Counts are non-negative integers (for count inputs); no HTSeq summary rows (lines beginning with '__') remain
- Output file exists and is parseable as CSV
- Row count equals the number of unique gene identifiers across all inputs

## Limitations

- Gene identifier columns must match across all files; if they do not, the merge will fail ('first column must contain the genes. If the gene columns do not match in all files, this tool will not work')
- Counts are not normalized or filtered — merging only assembles the matrix; filtering and normalization happen downstream (e.g. in DESeq2Shiny)
- HTSeq trailing summary-statistics rows must be removed or they will pollute the merged matrix

## Evidence

- [readme] readme_gene_count_merger_purpose: [evidence span withheld — anchor status: not_found]
- [readme] readme_gene_column_note: [evidence span withheld — anchor status: external_doc]
- [methods] methods_merging_functionality: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
