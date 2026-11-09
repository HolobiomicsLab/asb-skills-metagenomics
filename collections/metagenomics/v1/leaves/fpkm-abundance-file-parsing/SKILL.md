---
name: fpkm-abundance-file-parsing
description: Use when the input consists of multiple per-sample gene abundance/FPKM
  files (e.g., Cufflinks-style tabular gene output) that must be aligned by gene identifier
  and merged into one samples-and-genes matrix for downstream differential expression
  analysis.
when_to_use_negative:
- Input is already a single merged gene-by-sample matrix (no per-sample files to join)
- Input is raw per-sample gene count files (e.g., htseq output) — use NASQAR's GeneCountMerger
  for counts, which itself warns 'If the gene columns do not match in all files, this
  tool will not work'
- Only one sample file is provided — no merge step is needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NASQAR
  role: Web platform whose preprocessing app (mergeFPKMs) performs the sample FPKM
    file merge and offers gene ID to gene name conversion
  repo: https://github.com/nasqar/NASQAR
- name: R
  role: 'Implementation language: readr/read.table for parsing per-sample FPKM files
    and dplyr::full_join for the full outer join on gene identifiers'
- name: DESeq2Shiny
  role: Downstream bulk RNA-seq app that consumes the merged FPKM matrix for exploratory
    analysis and differential expression
  repo: https://github.com/nasqar/deseq2shiny
- name: GeneCountMerger
  role: Analogous NASQAR preprocessing tool that merges individual gene count files,
    defining the matching-gene-columns constraint the same merge step enforces
  repo: https://github.com/nasqar/GeneCountMerger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:7159ccaeb17e9addd9cb1369943d312ddcecd86fa327033ff24affcdfd3ad51c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/NASQAR
  tools:
  - DESeq2Shiny
  - GeneCountMerger
  - NASQAR
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fpkm-abundance-file-parsing

## Summary

Parses and merges per-sample gene FPKM/abundance files (Cufflinks-style tabular gene output) into a single genes-by-samples CSV matrix, as done by NASQAR's preprocessing apps, so the merged matrix can feed downstream RNA-seq tools like DESeq2Shiny.

## When to use

Use when the input consists of multiple per-sample gene abundance/FPKM files (e.g., Cufflinks-style tabular gene output) that must be aligned by gene identifier and merged into one samples-and-genes matrix for downstream differential expression analysis.

## When NOT to use

- Input is already a single merged gene-by-sample matrix (no per-sample files to join)
- Input is raw per-sample gene count files (e.g., htseq output) — use NASQAR's GeneCountMerger for counts, which itself warns 'If the gene columns do not match in all files, this tool will not work'
- Only one sample file is provided — no merge step is needed

## Inputs

- Per-sample Cufflinks-style tabular gene abundance/FPKM files (gene identifier as first column)
- List of sample file paths or an uploaded multi-sample file

## Outputs

- merged_fpkm_matrix.csv — genes-by-samples matrix of FPKM values with gene identifier as the first column and one column per input sample

## How to apply

Load each per-sample FPKM file with R's readr/read.table, keying rows on the gene identifier column. Merge all samples by full outer join on the gene identifier (e.g., dplyr::full_join) so genes absent from a given sample become NA or 0. Optionally convert gene IDs to gene names, a function NASQAR's preprocessing apps explicitly offer. Write the result to merged_fpkm_matrix.csv with the gene identifier as the first column, one column per input sample. The merged matrix must be directly consumable by NASQAR's downstream bulk RNA-seq apps such as DESeq2Shiny, which wraps DESeq2. This mirrors the analogous GeneCountMerger preprocessing tool, whose constraint is that the first column must contain the genes and gene columns must match across files.

## Related tools

- **NASQAR** (Web platform whose preprocessing app (mergeFPKMs) performs the sample FPKM file merge and offers gene ID to gene name conversion) — https://github.com/nasqar/NASQAR
- **R** (Implementation language: readr/read.table for parsing per-sample FPKM files and dplyr::full_join for the full outer join on gene identifiers)
- **DESeq2Shiny** (Downstream bulk RNA-seq app that consumes the merged FPKM matrix for exploratory analysis and differential expression) — https://github.com/nasqar/deseq2shiny
- **GeneCountMerger** (Analogous NASQAR preprocessing tool that merges individual gene count files, defining the matching-gene-columns constraint the same merge step enforces) — https://github.com/nasqar/GeneCountMerger

## Examples

```
merged <- Reduce(function(x,y) dplyr::full_join(x, y, by='gene_id'), lapply(files, readr::read_tsv)); write.csv(merged, 'merged_fpkm_matrix.csv', row.names=FALSE)
```

## Evaluation signals

- merged_fpkm_matrix.csv exists and is parseable as CSV
- Row count of the merged matrix equals the number of unique gene identifiers across all input sample files
- Matrix contains exactly one data column per input sample plus the gene identifier first column
- Genes absent from a given sample are filled with NA or 0 rather than dropped
- File loads directly into DESeq2Shiny without schema errors

## Limitations

- First column must contain the genes; if gene columns do not match across all files, the merge tool will not work (per GeneCountMerger README)
- The published NASQAR article documents merging gene counts and gene ID conversion generically; mergeFPKMs FPKM-specific behavior is not detailed in the paper itself
- NASQAR's modular design decouples preprocessing from downstream analysis, so the user must chain merge output into DESeq2Shiny manually rather than within one end-to-end workflow

## Evidence

- [methods] merging gene counts with gene ID conversion in NASQAR preprocessing: "Functionality includes merging gene counts, conversion of gene IDs to gene names"
- [readme] GeneCountMerger constraint on merging per-sample files: [evidence span withheld — anchor status: not_found]
- [readme] DESeq2Shiny as downstream consumer of merged matrices: [evidence span withheld — anchor status: external_doc]
- [abstract] NASQAR as the hosting platform for preprocessing apps: "we have created a dynamic web-based platform, NASQAR (Nucleic Acid SeQuence Analysis Resource)"
