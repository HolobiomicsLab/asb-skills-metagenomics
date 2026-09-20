---
name: source-associated-genome-database-handling
description: Use when assigning short-read (e.g. 2 × 150 bp Illumina NovaSeq 6000)
  shotgun metagenomic reads from water samples to a curated genome database with known
  source associations, to quantify fecal signal per source category.
when_to_use_negative:
- When potential fecal sources can and should be sequenced directly and a bespoke,
  sample-specific reference database is preferred over curated genome-to-source associations
- When reads are long-read or not short paired-end metagenomic reads — SourceApp is
  designed for short-read metagenomes and the bwa mem short-read mapping path
- When only contig-level abundance (not per-genome, source-associated assignment)
  is needed, or the input is already a feature/abundance table rather than raw reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bwa
  role: Indexes the source-associated genome database (bwa index) and aligns trimmed
    paired-end metagenomic reads to it with the bwa mem algorithm
- name: CoverM
  role: Summarizes per-genome sequence depth and number of reads mapped to genomes
    in each source category (v0.7.0), writing the per-genome counts table
  repo: https://github.com/wwood/CoverM
- name: fastp
  role: Quality-checks and trims reads to remove leftover adapter sequences and low-quality
    reads prior to mapping
- name: SourceApp
  role: 'Automates the workflow: supplies the default genome database and applies
    read-to-genome assignment thresholds (93% identity, 70% query coverage, 0% masking,
    LOD 0.01%) to infer source attribution'
  repo: https://github.com/bglindner/SourceApp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:e3f741a2b3eb9948c6d35cf0b68ddc4d3224843cf446073245b61ffe1d380ae2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wwood/CoverM
  tools:
  - CoverM
  - SourceApp
  - bwa
  - bwa (mem)
  - fastp
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# source-associated-genome-database-handling

## Summary

Maps quality-trimmed shotgun metagenomic reads to a source-associated genome database with bwa mem and summarizes per-genome mapped read counts and sequence depth by source category with CoverM, forming the read-mapping core of SourceApp's fecal source tracking (FST) framework.

## When to use

Use when assigning short-read (e.g. 2 × 150 bp Illumina NovaSeq 6000) shotgun metagenomic reads from water samples to a curated genome database with known source associations, to quantify fecal signal per source category.

## When NOT to use

- When potential fecal sources can and should be sequenced directly and a bespoke, sample-specific reference database is preferred over curated genome-to-source associations
- When reads are long-read or not short paired-end metagenomic reads — SourceApp is designed for short-read metagenomes and the bwa mem short-read mapping path
- When only contig-level abundance (not per-genome, source-associated assignment) is needed, or the input is already a feature/abundance table rather than raw reads

## Inputs

- gzipped raw shotgun metagenomic reads in FASTQ format (paired-end, e.g. from Illumina NovaSeq 6000 2 × 150-bp sequencing of mesocosms or waterway samples; mesocosm data under NCBI BioProject PRJNA1161616)
- source-associated genome database as reference FASTA files with known source categories (SourceApp default database from Zenodo 10.5281/zenodo.12795076, or a de novo database built with sourceapp_build.py)

## Outputs

- tab-separated per-genome counts table with one row per database genome, reporting mapped read counts and sequence depth values grouped by source category as computed by CoverM
- BAM alignments of metagenomic reads against the genome database (intermediate from bwa mem/CoverM)
- per-source attribution/apportionment estimates (prokaryotic cell fraction-like metrics when --use-geq is used by SourceApp)

## How to apply

First quality-check and trim the raw gzipped FASTQ reads with fastp to remove leftover adapter sequences and low-quality reads, optionally normalizing with bbnorm (target=30, min=5). Index every genome in the source-associated database (downloaded from Zenodo DOI 10.5281/zenodo.12795076 or built with sourceapp_build.py) using bwa index, then align the trimmed paired-end reads with bwa mem. Summarize alignments per genome with CoverM v0.7.0 to produce a tab-separated table of mapped read counts and sequence depth per genome grouped by source category. Apply SourceApp's read-to-genome assignment thresholds — 93% minimum read-to-genome alignment identity (-r/--percent-identity 0.93), 70% minimum alignment-to-read query coverage (-q/--query-coverage 0.7), and 0% target genome masking — which parameter tuning showed yield ~0.90 sensitivity and specificity. The rationale is that reads attributable to source-associated genomes approximate cell fractions of each fecal source when normalized by genome equivalents.

## Related tools

- **bwa** (Indexes the source-associated genome database (bwa index) and aligns trimmed paired-end metagenomic reads to it with the bwa mem algorithm)
- **CoverM** (Summarizes per-genome sequence depth and number of reads mapped to genomes in each source category (v0.7.0), writing the per-genome counts table) — https://github.com/wwood/CoverM
- **fastp** (Quality-checks and trims reads to remove leftover adapter sequences and low-quality reads prior to mapping)
- **SourceApp** (Automates the workflow: supplies the default genome database and applies read-to-genome assignment thresholds (93% identity, 70% query coverage, 0% masking, LOD 0.01%) to infer source attribution) — https://github.com/bglindner/SourceApp

## Examples

```
coverm genome --mapper bwa-mem -r combined_genomes.fna -1 trimmed_reads_1.fq.gz -2 trimmed_reads_2.fq.gz -t per_genome_counts.tsv --min-read-aligned-percent-identity 0.93 --min-read-aligned-base-pairs 0.7
```

## Evaluation signals

- The CoverM output table exists, is tab-separated, and contains exactly one row per genome in the database
- The table reports both mapped read counts and sequence depth values for each genome, grouped by source category
- Read-to-genome assignments respect the tuned thresholds: 93% minimum alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking
- Sources below the limit of detection (0.01% relative abundance / cell fraction) are not called as present
- Per-genome coverage values are plausible relative to input read depth (e.g. consistent with bbnorm normalization to target=30, min=5 if applied)

## Limitations

- SourceApp works best when users provide supplementary reference genomes to bolster the default database; performance varied substantially by source, with cows underestimated due to undersampling of cow fecal microbiomes
- Cross-reactive genome pairs (ANI ≥ 95% across source categories) are flagged because reads can be ambiguously attributed between closely related genomes
- SourceApp portions do not represent a mass balance but estimates of cell proportions based on genome equivalents, and cannot infer contamination age or nonconservative (growth-capable) populations

## Evidence

- [methods] fastp trimming before mapping: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [results] optimal assignment parameters: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] CoverM role and version: "CoverM (v0.7.0; https://github.com/wwood/CoverM)"
- [abstract] SourceApp mapping-based design: "created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases"
- [readme] CoverM purpose: [evidence span withheld — anchor status: external_doc]
- [readme] database supplement recommendation: "SourceApp works best when the user is able to provide supplementary reference genomes to bolster SourceApp's default genomic reference database."
