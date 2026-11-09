---
name: metagenomic-read-to-protein-alignment
description: Use when quality-filtered (host-filtered, duplicate-filtered, quality-trimmed)
  shotgun metagenomic FASTQs must be mapped to a microbial protein gene database like
  UniRef90 to obtain read-to-gene matches for functional profiling.
when_to_use_negative:
- Input reads have not undergone host read filtering, duplicate removal, or quality
  trimming — MetaLAFFA requires these quality control steps before the read mapping
  phase
- The analysis requires nucleotide-to-nucleotide or contig/gene-family-level alignment
  rather than 6-frame translated read-to-protein alignment
- The goal is assembling microbial genomes; the article notes assembly is prohibitively
  time and memory intensive and read-based annotation is the alternative pursued here
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: DIAMOND
  role: Aligner run in translated-alignment mode against the UniRef90 DIAMOND database,
    with --block_size 36 and --index_chunks 1 defaults, emitting tabular (m8) hit
    tables
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MetaLAFFA
  role: Snakemake pipeline whose read mapping phase invokes DIAMOND per mate FASTQ
    and passes hits to best-e-value filtering and fractional gene counting
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:354027634b28b973f82f98e4035c56e912abd484649d3df0512e9e6256848f91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - DIAMOND
  - MetaLAFFA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metagenomic-read-to-protein-alignment

## Summary

Align quality-controlled shotgun metagenomic reads to a protein-sequence gene database (UniRef90) in translated-alignment mode using DIAMOND, as done in MetaLAFFA's read mapping phase, to produce a read-to-gene hit table that feeds best-hit filtering and gene abundance counting.

## When to use

Use when quality-filtered (host-filtered, duplicate-filtered, quality-trimmed) shotgun metagenomic FASTQs must be mapped to a microbial protein gene database like UniRef90 to obtain read-to-gene matches for functional profiling.

## When NOT to use

- Input reads have not undergone host read filtering, duplicate removal, or quality trimming — MetaLAFFA requires these quality control steps before the read mapping phase
- The analysis requires nucleotide-to-nucleotide or contig/gene-family-level alignment rather than 6-frame translated read-to-protein alignment
- The goal is assembling microbial genomes; the article notes assembly is prohibitively time and memory intensive and read-based annotation is the alternative pursued here

## Inputs

- Quality-filtered gzipped FASTQ files per sample (forward, reverse, and singleton mates) named <sample_id>.<type_id>.fastq.gz
- Prepared UniRef90 DIAMOND database file (<name_of_database>.dmnd)

## Outputs

- DIAMOND tabular alignment output (m8-style hit table) per input FASTQ, retaining read-to-gene matches for best-hit filtering

## How to apply

Load the quality-filtered, gzipped FASTQ files per sample following MetaLAFFA's naming convention (<sample_id>.<type_id>.fastq.gz for R1, R2, and singleton mates). Run DIAMOND in translated-alignment mode (6-frame translated search of nucleotide reads against protein sequences) against the prepared UniRef90 DIAMOND database (<name_of_database>.dmnd). Follow MetaLAFFA defaults of --block_size 36 and --index_chunks 1 with 22 cores and ~220G RAM; reduce block_size/index_chunks when memory is limited since they control how much of the reference is loaded at a time. Align each mate FASTQ (R1, R2, singleton) separately — as in HUMAnN2 — to avoid overly penalizing read pairs where only one mate aligns to a gene. Emit the DIAMAND tabular (m8-style) hit table per input FASTQ, which downstream steps reduce to the best e-value hit per read before fractional gene counting.

## Related tools

- **DIAMOND** (Aligner run in translated-alignment mode against the UniRef90 DIAMOND database, with --block_size 36 and --index_chunks 1 defaults, emitting tabular (m8) hit tables) — https://github.com/borenstein-lab/MetaLAFFA
- **MetaLAFFA** (Snakemake pipeline whose read mapping phase invokes DIAMOND per mate FASTQ and passes hits to best-e-value filtering and fractional gene counting) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
diamond blastx -d uniref90.dmnd -q SRS011061.R1.fastq.gz -o hits.m8 --block-size 36 --index-chunks 1 -p 22
```

## Evaluation signals

- A hits output file exists for every input FASTQ (R1, R2, and singleton) of every sample
- Each hits output is parseable as a tab-separated DIAMOND tabulation (m8-style hit table)
- Mapped-read fraction is consistent with the paper's reported 53%–78% UniRef90 match range across samples
- Each mate FASTQ was aligned independently (no merged pair alignment), matching MetaLAFFA's per-FASTQ-type processing before downstream concatenation
- DIAMOND parameters match configured values (--block_size 36, --index_chunks 1, 22 cores) or are consciously reduced when memory is limited

## Limitations

- DIAMOND memory footprint is controlled by --block_size and --index_chunks (~220G RAM at defaults); memory-limited machines must reduce these, which affects how much of the reference is loaded at a time
- Read pairs are aligned as separate mates, so information about concordant paired alignments is not used during mapping
- UniRef90 matching rates vary widely (53%–78% across samples), so many reads will remain unmapped and contribute no gene counts

## Evidence

- [intro] aligning reads to the UniRef90 database of protein sequences using DIAMOND: "aligning reads to the UniRef90 database of protein sequences using DIAMOND"
- [intro] the read mapping phase aligns the quality-controlled reads to a sequence database of microbial genes and calculates the abundance of each gene: "the read mapping phase aligns the quality-controlled reads to a sequence database of microbial genes and calculates the abundance of each gene"
- [methods] keeping only hits with the best e-value for each read: "keeping only hits with the best e-value for each read and allowing best hits to include genes without functional annotations."
- [results] The percentage of reads that had a match in the UniRef90 database varied from 53% to 78% across samples: "The percentage of reads that had a match in the UniRef90 [21] database varied from 53% to 78% across samples"
- [readme] Download and prepare the UniRef90 database for read mapping and functional annotation.: "`-u`: Download and prepare the UniRef90 database for read mapping and functional annotation."
