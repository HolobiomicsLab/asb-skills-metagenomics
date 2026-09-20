---
name: gene-length-normalization
description: Reference for computing per-gene abundances from read-to-gene alignment
  hits when reads may match multiple genes equally well, and when gene-length bias
  in shotgun metagenomic count data must be corrected.
when_to_use_negative:
- Input is already a gene abundance table — this skill operates on raw read-to-gene
  alignment hits, not on existing feature tables
- Aggregating gene abundances into orthology groups or pathways (this is MetaLAFFA's
  separate ortholog-counting and aggregation steps)
- When alignments retain only a single unambiguous best hit per read, fractional tie
  distribution reduces to plain integer counting
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: MetaLAFFA
  role: Pipeline whose gene-counting step performs fractional counting and gene length
    normalization
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom Python script performs gene length normalization and fractional counting
- name: DIAMOND
  role: Produces the read-to-gene alignment hits (UniRef90) that feed the counting
    step
  repo: https://github.com/bbuchfink/diamond_docs
- name: Snakemake
  role: Wraps the counting step in the reproducible, cluster-capable pipeline
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:3d0d3ff05c7816fc7e8cc2ffa16d0e0235f9aa668efaab444c963c162e3e271b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gene-length-normalization

## Summary

MetaLAFFA's gene abundance calculation: per-gene read counts are computed with fractional counting for reads tied across multiple equally-good hits, then normalized by gene length so longer genes are not over-counted. This is implemented in a custom Python script that consumes hit-filtered DIAMOND alignments and a gene-length normalization (.norm) table.

## When to use

Reference for computing per-gene abundances from read-to-gene alignment hits when reads may match multiple genes equally well, and when gene-length bias in shotgun metagenomic count data must be corrected.

## When NOT to use

- Input is already a gene abundance table — this skill operates on raw read-to-gene alignment hits, not on existing feature tables
- Aggregating gene abundances into orthology groups or pathways (this is MetaLAFFA's separate ortholog-counting and aggregation steps)
- When alignments retain only a single unambiguous best hit per read, fractional tie distribution reduces to plain integer counting

## Inputs

- Hit-filtered DIAMOND alignment output (gzipped, best e-value hits per read) per sample
- Gene length normalization table (.norm file) with each gene's length and normalization factor (gene length / average gene length)

## Outputs

- Per-sample gene abundance table (tab-separated; genes as rows, samples as columns) with fractional, length-normalized counts

## How to apply

Load the hit-filtered DIAMOND alignment output (gzipped best e-value hits per read) for each sample, and the .norm file giving each gene's length and its normalization factor (gene length / average gene length). Sum retained hits per gene, dividing each multi-mapping read's count of 1 equally among all genes it ties with (MetaLAFFA uses fractional gene counting by default). Then divide each gene's fractional read count by its length normalization factor, since counts of reads mapping to a gene should be normalized by the length of the gene. Assemble per-sample counts into a single tab-separated table with genes as rows and samples as columns.

## Related tools

- **MetaLAFFA** (Pipeline whose gene-counting step performs fractional counting and gene length normalization) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom Python script performs gene length normalization and fractional counting)
- **DIAMOND** (Produces the read-to-gene alignment hits (UniRef90) that feed the counting step) — https://github.com/bbuchfink/diamond_docs
- **Snakemake** (Wraps the counting step in the reproducible, cluster-capable pipeline) — https://github.com/borenstein-lab/MetaLAFFA

## Evaluation signals

- Output TSV exists and is tab-delimited with one row per UniRef90 gene and one column per sample
- Each sample's total counts are consistent with fractional (not double-counted) read contributions — a read tied across N genes contributes 1/N to each
- Gene counts are reduced by each gene's length normalization factor (gene length / average gene length) relative to raw fractional counts
- Counts derive only from retained best e-value hits, matching the hit-filtered DIAMOND output

## Limitations

- Gene length normalization alone does not correct for other library-size differences between samples; downstream normalization may still be needed
- Fractional counting applies only to reads that match multiple genes equally well; hits with worse e-values are discarded during best-hit filtering before counting
- Reads without a match in the reference database (e.g. only 53–78% of reads matched UniRef90 across tested samples) contribute nothing to gene abundances

## Evidence

- [intro] gene abundance calculation with fractional counts for multi-matches: "gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [methods] fractional counting implemented in a custom Python script: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [methods] rationale for gene length normalization: "the counts of reads mapping to a gene should be normalized by the length of the gene"
- [readme] pipeline context for abundance calculation: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups"
