---
name: gene-identifier-alignment-across-samples
description: Use when you have one tabular gene abundance file per sample (FPKM or
  count files with genes in the first column) and need a single merged csv matrix
  keyed on gene identifiers for downstream bulk RNA-seq analysis.
when_to_use_negative:
- Gene columns differ across the input files (GeneCountMerger explicitly states 'If
  the gene columns do not match in all files, this tool will not work') — harmonize
  identifiers first
- Input is already a single merged genes-by-samples matrix, e.g., DESeq2Shiny is being
  fed directly
- Sample files use transcript- or isoform-level identifiers rather than gene identifiers
  — alignment on gene id would mismatch rows
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: NASQAR
  role: Web-based platform hosting the mergeFPKMs preprocessing app that performs
    the gene-identifier-keyed merge of sample files
  repo: https://github.com/nasqar/NASQAR
- name: R
  role: Implementation language; merge performed with base R/dplyr full_join on the
    gene identifier column
- name: GeneCountMerger
  role: NASQAR preprocessing tool for merging individual gene count files; same identifier-alignment
    principle applied to count (htseq-style) inputs
  repo: https://github.com/nasqar/GeneCountMerger
- name: DESeq2Shiny
  role: Downstream NASQAR bulk RNA-seq app that consumes the merged matrix produced
    by this skill
  repo: https://github.com/nasqar/deseq2shiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:892950cbdb045a84873d40ecef058a53106c245e0d9058685401701652b84013
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - DESeq2Shiny
  - GeneCountMerger
  - NASQAR
  - R
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-identifier-alignment-across-samples

## Summary

Aligns per-sample gene abundance files (e.g., Cufflinks-style FPKM outputs) on their shared gene identifier column and merges them into a single genes-by-samples matrix, as implemented in NASQAR's mergeFPKMs/GeneCountMerger preprocessing apps so downstream RNA-seq apps like DESeq2Shiny can consume one unified table.

## When to use

Use when you have one tabular gene abundance file per sample (FPKM or count files with genes in the first column) and need a single merged csv matrix keyed on gene identifiers for downstream bulk RNA-seq analysis.

## When NOT to use

- Gene columns differ across the input files (GeneCountMerger explicitly states 'If the gene columns do not match in all files, this tool will not work') — harmonize identifiers first
- Input is already a single merged genes-by-samples matrix, e.g., DESeq2Shiny is being fed directly
- Sample files use transcript- or isoform-level identifiers rather than gene identifiers — alignment on gene id would mismatch rows

## Inputs

- Per-sample gene FPKM files (Cufflinks-style tabular gene abundance output), one file per sample
- Gene identifier column present as the first column in each input file

## Outputs

- A single csv matrix (e.g., merged_fpkm_matrix.csv) of samples vs. gene FPKM values, gene identifier in the first column, one column per input sample

## How to apply

Load each per-sample FPKM/count file with R's readr or read.table, then key each file's rows on its gene identifier column (gene id and/or gene name). Merge all samples by full outer join on the gene identifier (e.g., dplyr::full_join) so genes present in some but not all samples are retained, filling absent values with NA or 0. Write the result to a csv (e.g., merged_fpkm_matrix.csv) with the gene identifier as the first column, one column per sample. Rationale: NASQAR decouples preprocessing from analysis, so this merged matrix is the handoff artifact for downstream apps such as DESeq2Shiny; the GeneCountMerger README warns the merge only works if the gene columns match across files, so identifier consistency must be checked before joining.

## Related tools

- **NASQAR** (Web-based platform hosting the mergeFPKMs preprocessing app that performs the gene-identifier-keyed merge of sample files) — https://github.com/nasqar/NASQAR
- **R** (Implementation language; merge performed with base R/dplyr full_join on the gene identifier column)
- **GeneCountMerger** (NASQAR preprocessing tool for merging individual gene count files; same identifier-alignment principle applied to count (htseq-style) inputs) — https://github.com/nasqar/GeneCountMerger
- **DESeq2Shiny** (Downstream NASQAR bulk RNA-seq app that consumes the merged matrix produced by this skill) — https://github.com/nasqar/deseq2shiny

## Examples

```
merged <- Reduce(function(x,y) dplyr::full_join(x, y, by = "gene_id"), sample_fpkm_list); write.csv(merged, "merged_fpkm_matrix.csv", row.names = FALSE)
```

## Evaluation signals

- merged_fpkm_matrix.csv exists and is parseable as csv
- Row count of the merged matrix equals the number of unique gene identifiers across all input sample files
- Merged matrix contains exactly one column per input sample (plus the gene identifier column)
- Genes absent from a given sample appear as NA (or 0) in that sample's column rather than being silently dropped
- The merged file loads without error into downstream NASQAR apps such as DESeq2Shiny

## Limitations

- If the gene columns do not match in all files, the merge tool will not work (GeneCountMerger README caveat)
- Genes missing from some samples introduce NA/0 fill values, which may require handling before differential expression analysis
- Requires per-sample inputs keyed on gene identifiers; no built-in cross-reference of incompatible identifier schemes (only gene ID-to-name conversion is offered)

## Evidence

- [methods] mergeFPKMs app description: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
- [intro] modular preprocessing scope: "By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently"
- [readme] GeneCountMerger gene-column requirement: [evidence span withheld — anchor status: external_doc]
- [readme] GeneCountMerger role: [evidence span withheld — anchor status: not_found]
