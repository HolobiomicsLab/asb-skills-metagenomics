---
name: shotgun-metagenome-read-qc-and-trimming
description: Use when raw shotgun metagenomic FASTQ reads (e.g., gzipped Illumina
  NovaSeq 6000 2x150-bp mesocosm reads) must be cleaned of adapters and low-quality
  sequence prior to genome mapping or abundance estimation.
when_to_use_negative:
- Input reads are already trimmed or supplied as BAM/sorted alignment files — fastp
  trimming would be redundant before CoverM, which can consume BAM directly
- The goal is long-read MAG recovery without short-read QC — filtlong (--min_length
  1000 --min_window_q 0.8 --window_size 250) and bbnorm are the appropriate filters,
  not paired-end fastp trimming
- The user only needs source attribution from a prebuilt database and is running the
  full sourceapp.py pipeline, which already performs the fastp quality-check step
  internally
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: fastp
  role: Quality-checks sequences, removing leftover adapter sequences and low-quality
    reads from shotgun metagenomic FASTQ files before mapping
- name: SourceApp
  role: Pipeline that consumes the gzipped raw FASTQ inputs and automates the QC/trimming
    step prior to read mapping against the source-associated genome database
  repo: github.com/bglindner/SourceApp
- name: CoverM
  role: 'Downstream consumer of trimmed reads: maps reads and summarizes per-genome
    mapped read counts and sequence depth (v0.7.0)'
  repo: github.com/wwood/CoverM
- name: bwa
  role: Indexes the source-associated genome database (bwa index) and maps trimmed
    paired-end reads with the bwa mem algorithm
- name: bbnorm
  role: BBTools read normalization of trimmed reads (target = 30, min = 5) in the
    database-construction branch of the workflow
- name: filtlong
  role: Long-read trimming/filtering (--min_length 1000 --min_window_q 0.8 --window_size
    250) for Nanopore reads used in MAG recovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:fca1ece72c82e845d8c0bde4fcd84dcd7ba25b55fcc4d3adc311ee6a8bf19192
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CoverM
  - SourceApp
  - bbnorm
  - bwa
  - bwa (mem)
  - fastp
  - filtlong
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# shotgun-metagenome-read-qc-and-trimming

## Summary

Quality-check and trim shotgun metagenomic FASTQ reads with fastp to remove leftover adapter sequences and low-quality reads before mapping reads to a source-associated genome database with bwa mem/CoverM, as performed in the SourceApp fecal source tracking pipeline.

## When to use

Use when raw shotgun metagenomic FASTQ reads (e.g., gzipped Illumina NovaSeq 6000 2x150-bp mesocosm reads) must be cleaned of adapters and low-quality sequence prior to genome mapping or abundance estimation.

## When NOT to use

- Input reads are already trimmed or supplied as BAM/sorted alignment files — fastp trimming would be redundant before CoverM, which can consume BAM directly
- The goal is long-read MAG recovery without short-read QC — filtlong (--min_length 1000 --min_window_q 0.8 --window_size 250) and bbnorm are the appropriate filters, not paired-end fastp trimming
- The user only needs source attribution from a prebuilt database and is running the full sourceapp.py pipeline, which already performs the fastp quality-check step internally

## Inputs

- Raw gzipped paired-end shotgun metagenomic FASTQ reads (e.g., Illumina NovaSeq 6000 2x150-bp reads from mesocosms, BioProject PRJNA1161616)
- Optional: trimmed ONT minION long-read FASTQ for MAG-recovery workflows (filtered with filtlong)

## Outputs

- Adapter-trimmed, quality-filtered paired-end FASTQ files ready for bwa mem mapping
- fastp QC report (implicit per 'quality checked using fastp')
- Downstream: per-genome CoverM v0.7.0 tab-separated counts table of mapped reads and sequence depth grouped by source category

## How to apply

Take the raw gzipped paired-end FASTQ files (in the SourceApp study, shotgun sequencing was done on an Illumina NovaSeq 6000 for 2x150-bp reads from mesocosms in NCBI BioProject PRJNA1161616) and run fastp to remove leftover adapter sequences and low-quality reads. The SourceApp pipeline expects users to provide gzipped raw reads in FASTQ format and automates this quality-check step internally, but it can be run explicitly as a standalone pre-processing stage. The trimmed paired-end output then feeds directly into downstream steps: indexing all genomes in the source-associated database (SourceApp default database from Zenodo 10.5281/zenodo.12795076) with bwa index, mapping with bwa mem, and per-genome depth/read-count summarization with CoverM v0.7.0. Note that within the broader SourceApp database-construction workflow, short reads may additionally be normalized with BBTools bbnorm (target = 30, min = 5), and long reads filtered with filtlong (--min_length 1000 --min_window_q 0.8 --window_size 250), depending on whether the reads feed MAG recovery or read-mapping-based source attribution. Validate success by confirming trimmed FASTQ outputs exist and that the subsequent CoverM per-genome counts table is produced with one tab-separated row per database genome.

## Related tools

- **fastp** (Quality-checks sequences, removing leftover adapter sequences and low-quality reads from shotgun metagenomic FASTQ files before mapping)
- **SourceApp** (Pipeline that consumes the gzipped raw FASTQ inputs and automates the QC/trimming step prior to read mapping against the source-associated genome database) — github.com/bglindner/SourceApp
- **CoverM** (Downstream consumer of trimmed reads: maps reads and summarizes per-genome mapped read counts and sequence depth (v0.7.0)) — github.com/wwood/CoverM
- **bwa** (Indexes the source-associated genome database (bwa index) and maps trimmed paired-end reads with the bwa mem algorithm)
- **bbnorm** (BBTools read normalization of trimmed reads (target = 30, min = 5) in the database-construction branch of the workflow)
- **filtlong** (Long-read trimming/filtering (--min_length 1000 --min_window_q 0.8 --window_size 250) for Nanopore reads used in MAG recovery)

## Evaluation signals

- Trimmed FASTQ outputs exist for every input pair and remain gzip-compressed paired-end files
- Adapter content and low-quality tail reads reported by fastp are reduced relative to raw input
- The subsequent CoverM output table exists and is tab-separated with exactly one row per database genome
- The CoverM table reports mapped read counts and sequence depth values for each genome grouped by source category
- bwa index completes on all genomes in the source-associated database before mapping, so bwa mem produces alignments for every trimmed sample

## Limitations

- Trimming removes low-quality reads, which can reduce depth on low-biomass samples; the SourceApp LOD of 0.01% relative abundance bounds detection of weak fecal signals
- SourceApp's default database underrepresents cow fecal microbiomes, so trimming/mapping quality cannot compensate for missing source genomes
- CoverM coverage estimates depend on upstream trim quality; over-aggressive trimming can drop reads and distort mapped read counts per genome

## Evidence

- [methods] fastp removes adapters and low-quality reads: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [methods] trimmed reads normalized with bbnorm target=30 min=5: [evidence span withheld — anchor status: not_found]
- [methods] filtlong long-read filter parameters: "trimmed with filtlong (--min_length 1000 --min_window_q 0.8 --window_size 250)"
- [readme] SourceApp expects gzipped raw FASTQ input and automates QC: [evidence span withheld — anchor status: not_found]
- [other] CoverM per-genome output validation for the mapping step fed by trimmed reads: [evidence span withheld — anchor status: not_found]
