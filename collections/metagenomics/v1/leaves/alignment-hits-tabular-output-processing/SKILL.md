---
name: alignment-hits-tabular-output-processing
description: 'Use when quality-filtered FASTQ shotgun metagenomic reads (MetaLAFFA
  naming: <sample_id>.<type_id>.fastq.gz) must be aligned to UniRef90 gene sequences
  and the DIAMOND tabular hits need processing for -hit selection and gene abundance
  computation.'
when_to_use_negative:
- Input is not raw shotgun metagenomic FASTQ but an already-computed gene abundance
  table or orthology-group profile (the annotation/counting steps downstream are already
  done).
- The goal is host-read removal (e.g. filtering human reads against hs37d5) — MetaLAFFA
  uses Bowtie 2 nucleotide mapping for that step, not DIAMOND translated alignment
  to UniRef90.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: DIAMOND
  role: Aligner run in translated-alignment mode against the UniRef90 .dmnd database
    to produce the tabular (m8) read-to-gene hits processed by this skill
  repo: bbuchfink/diamond_docs
- name: MetaLAFFA
  role: Pipeline (implemented as a Snakemake workflow) that configures DIAMOND, runs
    mapping per FASTQ mate, filters hits to best e-value, and computes fractional
    gene counts
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:2c6f39e69c810e93af225bc65394ec11d42cb10d974ac65de88aba8a62ce7cd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond_docs
  tools:
  - DIAMOND
  - MetaLAFFA
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# alignment-hits-tabular-output-processing

## Summary

Maps quality-controlled shotgun metagenomic reads to the UniRef90 protein database with DIAMOND in translated-alignment mode and processes the resulting m8-style tabular hit tables into per-FASTQ read-to-gene match files for downstream best-hit filtering and gene counting.

## When to use

Use when quality-filtered FASTQ shotgun metagenomic reads (MetaLAFFA naming: <sample_id>.<type_id>.fastq.gz) must be aligned to UniRef90 gene sequences and the DIAMOND tabular hits need processing for best-hit selection and gene abundance computation.

## When NOT to use

- Input is not raw shotgun metagenomic FASTQ but an already-computed gene abundance table or orthology-group profile (the annotation/counting steps downstream are already done).
- The goal is host-read removal (e.g. filtering human reads against hs37d5) — MetaLAFFA uses Bowtie 2 nucleotide mapping for that step, not DIAMOND translated alignment to UniRef90.

## Inputs

- Quality-filtered, gzipped FASTQ files per sample following MetaLAFFA naming (<sample_id>.<type_id>.fastq.gz; R1, R2, and singleton mates)
- Prepared UniRef90 DIAMOND database file (<name_of_database>.dmnd)
- DIAMOND parameters (--block_size, --index_chunks, thread/memory limits)

## Outputs

- Per-FASTQ DIAMOND tabular alignment output (m8-style hit table) of read-to-gene matches
- Best-hit-filtered read-to-gene mapping (best e-value hits per read, ties allowed)
- Gene abundance counts with fractional counts for multi-matches and gene length normalization

## How to apply

Load the quality-filtered, gzipped FASTQ files for each sample and run DIAMOND against the prepared UniRef90 DIAMOND database (<name_of_database>.dmnd) so reads are 6-frame translated and aligned to protein sequences. Configure DIAMOND with MetaLAFFA defaults --block_size 36 and --index_chunks 1 (requesting ~22 cores and ~220G RAM; reduce block_size/index_chunks if memory-limited, since these control how much of the reference is loaded at a time). Align each mate FASTQ (R1, R2, singleton) separately — as in HUMAnN2 — to avoid overly penalizing read pairs where only one mate aligns to a gene, then concatenate downstream. Write the DIAMOND tabular (m8) alignment output per input FASTQ, then refine by scanning the matches to identify the best match (or tied best matches) per read, keeping only hits with the best e-value and allowing best hits to include genes without functional annotations. Finally compute gene abundances via fractional counting (each read matching multiple equally good genes contributes a fractional count), with gene length normalization, so a mapped-read fraction consistent with the paper's reported 53%-78% UniRef90 range is expected.

## Related tools

- **DIAMOND** (Aligner run in translated-alignment mode against the UniRef90 .dmnd database to produce the tabular (m8) read-to-gene hits processed by this skill) — bbuchfink/diamond_docs
- **MetaLAFFA** (Pipeline (implemented as a Snakemake workflow) that configures DIAMOND, runs mapping per FASTQ mate, filters hits to best e-value, and computes fractional gene counts) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
diamond blastx -d UniRef90.dmnd -q <sample_id>.R1.fastq.gz -o <sample_id>.R1.hits.m8 --block-size 36 --index-chunks 1 --threads 22
```

## Evaluation signals

- A hits output exists for every input FASTQ (R1, R2, singleton) and each file is parseable as a tab-separated DIAMOND tabulation (m8-style).
- The mapped-read fraction per sample falls within the paper's reported 53%-78% UniRef90 match range.
- Best-hit filtering retains only the best e-value hit(s) per read, with ties preserved and hits to genes lacking functional annotations allowed.
- Genes matched equally well by a read receive fractional counts rather than full counts, and gene counts are length-normalized.
- Per-pair alignment is not penalized: each mate was aligned separately before downstream concatenation, consistent with the HUMAnN2-style strategy.

## Limitations

- DIAMOND's --block_size and --index_chunks control how much of the reference is loaded at a time; defaults (36 and 1) require ~220G RAM and 22 cores, and memory-limited machines must lower them.
- Reads with no match in UniRef90 are lost at this step; across the tested HMP stool samples 22%-47% of reads had no database match.
- Preparing the UniRef90 database is resource intensive (several hours, ~40GB RAM, ~150-300GB free disk space).
- MetaLAFFA is supported on Mac and Linux only, not Windows.

## Evidence

- [intro] Read mapping phase: align quality-controlled reads to microbial gene sequence database and calculate gene abundances: "the read mapping phase aligns the quality-controlled reads to a sequence database of microbial genes and calculates the abundance of each gene"
- [intro] DIAMOND against UniRef90: "aligning reads to the UniRef90 database of protein sequences using DIAMOND"
- [intro] Best match refinement: "MetaLAFFA scans these matches to identify the best match (or best matches if there are ties) for each read in the database"
- [results] Reported mapping range: "The percentage of reads that had a match in the UniRef90 [21] database varied from 53% to 78% across samples"
- [readme] UniRef90 database preparation: "-u`: Download and prepare the UniRef90 database for read mapping and functional annotation."
- [readme] DIAMOND core configuration: "changing the number of cores defined in `cluster_params` to be the number of cores you want a single DIAMOND alignment process to use"
