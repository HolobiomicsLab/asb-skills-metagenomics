---
name: metagenomic-hit-summarization
description: Use when you have filtered read-to-gene DIAMOND alignment hits ( e-value
  per read) from shotgun metagenomic data and need to compute per-gene abundance counts,
  including fractional assignment of multi-mapping reads.
when_to_use_negative:
- Input is already a gene abundance table rather than raw read-to-gene alignment hits
- Read-to-gene alignments have not yet been filtered to best e-value hits per read
- The goal is read-based taxonomic profiling rather than gene-centric functional abundance
  counting
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaLAFFA
  role: Pipeline whose gene-counting step performs fractional counting and gene length
    normalization via a custom Python script
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: DIAMOND
  role: Produces the read-to-gene alignments whose best e-value hits are summed into
    gene abundances
- name: Snakemake
  role: Implements the MetaLAFFA gene-counting step as a pipeline stage
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom Python script performing fractional gene counting and gene length normalization
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:2de76972b2aff92c1b2d15be57f160244bf614b1ee435d031b2c2427745d37d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# metagenomic-hit-summarization

## Summary

MetaLAFFA-style per-gene abundance counting: retained best-hit read-to-gene alignments are converted into gene abundance counts, with reads matching multiple genes equally well contributing fractional counts split evenly, then normalized by gene length.

## When to use

Use when you have filtered read-to-gene DIAMOND alignment hits (best e-value per read) from shotgun metagenomic data and need to compute per-gene abundance counts, including fractional assignment of multi-mapping reads.

## When NOT to use

- Input is already a gene abundance table rather than raw read-to-gene alignment hits
- Read-to-gene alignments have not yet been filtered to best e-value hits per read
- The goal is read-based taxonomic profiling rather than gene-centric functional abundance counting

## Inputs

- Hit-filtered DIAMOND alignment output (best e-value hits per read, gzipped) per sample
- Gene length normalization table (.norm file) with each gene's length and normalization factor (gene length / average gene length)

## Outputs

- Per-sample gene abundance counts (fractional, length-normalized)
- Combined tab-separated abundance table with genes as rows and samples as columns

## How to apply

Load the hit-filtered DIAMOND alignment output (gzipped best e-value hits per read) for each sample, plus the gene length normalization table (.norm file) whose factor is gene length divided by average gene length. Sum retained hits per gene with a custom Python script implementing fractional counting: a read mapping equally well to N genes contributes 1/N to each gene's abundance, preventing double counting. Divide each gene's fractional read count by its length normalization factor, since longer genes are more likely to be sequenced and would otherwise be overcounted. Assemble per-sample counts into a tab-separated abundance table with genes (e.g. UniRef90 IDs) as rows and samples as columns.

## Related tools

- **MetaLAFFA** (Pipeline whose gene-counting step performs fractional counting and gene length normalization via a custom Python script) — https://github.com/borenstein-lab/MetaLAFFA
- **DIAMOND** (Produces the read-to-gene alignments whose best e-value hits are summed into gene abundances)
- **Snakemake** (Implements the MetaLAFFA gene-counting step as a pipeline stage) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom Python script performing fractional gene counting and gene length normalization) — https://github.com/borenstein-lab/MetaLAFFA

## Evaluation signals

- Output TSV exists and is tab-delimited with one row per UniRef90 gene
- Each sample's counts sum consistently with fractional (not double-counted) read contributions — a read tied across N genes contributes 1/N to each
- Counts are length-normalized: each gene's fractional count is divided by its .norm factor (gene length / average gene length)
- Abundance table has samples as columns and genes as rows, matching all processed samples

## Limitations

- Fractional counting splits reads evenly among equally-best gene hits; this default assumes ties should be distributed uniformly
- Counts must be normalized by gene length because longer genes are more likely to be sequenced
- Gene counting depends on upstream best e-value hit filtering; unfiltered alignments will overcount

## Evidence

- [methods] Fractional gene counting with length normalization: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [methods] Gene length normalization rationale: "the counts of reads mapping to a gene should be normalized by the length of the gene"
- [intro] Fractional counts for multi-matches: "gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [intro] Read mapping phase abundance calculation: "the read mapping phase aligns the quality-controlled reads to a sequence database of microbial genes and calculates the abundance of each gene"
