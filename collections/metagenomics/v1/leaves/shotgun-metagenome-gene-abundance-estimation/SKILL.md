---
name: shotgun-metagenome-gene-abundance-estimation
description: Use when converting per-read DIAMOND alignments ( e-value hits per read)
  against a UniRef90-style gene catalog into a gene-by-sample abundance table, especially
  when reads tie across multiple equally good gene hits.
when_to_use_negative:
- Input is already a gene-by-sample abundance table (proceed directly to ortholog
  counting/aggregation)
- You need strictly integer read counts per gene with no handling of equally well-mapping
  multi-hits (fractional counting would be inappropriate)
- Input reads have not passed MetaLAFFA's quality control phase (host filtering, duplicate
  removal, Trimmomatic MAXINFO trimming), since unfiltered hits skew counts
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaLAFFA
  role: Pipeline whose gene counting step performs fractional counting and gene length
    normalization via a custom Python script
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom Python script implements the fractional counting and length normalization
    computation
  repo: https://github.com/engal/MetaLAFFA
- name: DIAMOND
  role: Produces the read-to-gene alignment hits (filtered to best e-value per read)
    that serve as counting input
- name: Snakemake
  role: Workflow engine that chains the gene counting step into the rest of the MetaLAFFA
    pipeline
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:80e44fb8519fe59432a16893b02c437d43eaf525ee061a8a96753ca7622f28de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# shotgun-metagenome-gene-abundance-estimation

## Summary

MetaLAFFA's gene counting step computes per-gene abundances from filtered DIAMOND read-to-gene alignments by summing retained best-e-value hits, assigning fractional counts (1/n) to genes tied in a read's best match, and dividing each gene's count by a length normalization factor (gene length / average gene length).

## When to use

Use when converting per-read DIAMOND alignments (best e-value hits per read) against a UniRef90-style gene catalog into a gene-by-sample abundance table, especially when reads tie across multiple equally good gene hits.

## When NOT to use

- Input is already a gene-by-sample abundance table (proceed directly to ortholog counting/aggregation)
- You need strictly integer read counts per gene with no handling of equally well-mapping multi-hits (fractional counting would be inappropriate)
- Input reads have not passed MetaLAFFA's quality control phase (host filtering, duplicate removal, Trimmomatic MAXINFO trimming), since unfiltered hits skew counts

## Inputs

- Hit-filtered DIAMOND alignment output (best e-value hits per read, gzipped) per sample
- Gene length normalization table (.norm file) with per-gene length and normalization factor (gene length / average gene length)

## Outputs

- Tab-separated gene abundance table (genes as rows, samples as columns) with length-normalized fractional counts per UniRef90 gene

## How to apply

Load the hit-filtered DIAMOND alignment output (gzipped best e-value hits per read) for each quality-controlled sample, plus the gene length normalization table (.norm file) where each factor equals gene length divided by the average gene length. For each read, identify its set of equally well-matching genes and distribute the read's count of 1 evenly across those genes; sum these fractional contributions per gene using a custom Python script implementing MetaLAFFA's fractional counting. Divide each gene's fractional read count by its length normalization factor, since longer genes are more likely to be sequenced and would otherwise be over-counted. Finally, assemble per-sample gene counts into a single tab-separated abundance table with UniRef90 genes as rows and samples as columns; this gene abundance table then feeds MetaLAFFA's ortholog counting and functional annotation phase.

## Related tools

- **MetaLAFFA** (Pipeline whose gene counting step performs fractional counting and gene length normalization via a custom Python script) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom Python script implements the fractional counting and length normalization computation) — https://github.com/engal/MetaLAFFA
- **DIAMOND** (Produces the read-to-gene alignment hits (filtered to best e-value per read) that serve as counting input)
- **Snakemake** (Workflow engine that chains the gene counting step into the rest of the MetaLAFFA pipeline) — https://github.com/borenstein-lab/MetaLAFFA

## Evaluation signals

- Output TSV exists, is tab-delimited, and contains one row per UniRef90 gene with samples as columns
- Each sample's total counts are consistent with fractional (not double-counted) read contributions — a read with n equally best hits adds 1/n to each tied gene
- Per-gene values reflect division by the .norm file's length normalization factor (gene length / average gene length)
- Gene counts can be traced back to the gzipped hit-filtered DIAMOND output for the same sample (auditability)
- Downstream ortholog counting/annotation consumes the table without schema errors

## Limitations

- Fractional counting spreads a read's weight evenly across tied genes, so abundances are non-integer and per-gene values are diluted for ambiguous reads
- Counts depend on gene length normalization; mis-specified gene lengths or a missing .norm file bias longer/shorter genes
- Only best-e-value hits per read (as retained by the hit-filtering step) are counted; reads with no UniRef90 match (53–78% match rate observed across samples) contribute nothing

## Evidence

- [methods] Fractional counting with length normalization: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [methods] Rationale for length normalization: "the counts of reads mapping to a gene should be normalized by the length of the gene"
- [intro] Fractional count definition for tied best matches: "gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [readme] Pipeline context of the gene counting step: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups"
