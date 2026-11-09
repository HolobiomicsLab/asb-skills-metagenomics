---
name: fecal-source-tracking-read-mapping
description: Use when assigning short-read shotgun metagenomic reads (e.g., 2 × 150-bp
  Illumina NovaSeq 6000) from waterway samples to curated genome databases with known
  source-associations to quantify fecal signal per source category.
when_to_use_negative:
- Input is long-read data or an already-assembled metagenome rather than short paired-end
  reads — SourceApp is designed to apportion fecal signal in short read metagenomes
- The goal is taxonomic profiling against a general database rather than attribution
  to genomes with known source-associations (e.g., fecal vs. lake water categories)
- Potential fecal sources must be distinguished solely from marker genes without reference
  genomes — this framework requires a genome-to-source-association database
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: Automates read mapping to source-associated genome databases, normalization
    by genome equivalents, and fecal source attribution/apportionment
  repo: https://github.com/bglindner/SourceApp
- name: bwa
  role: Indexes the genome database (bwa index) and maps trimmed paired-end reads
    to it with the mem algorithm
- name: CoverM
  role: Summarizes per-genome sequence depth and mapped read counts per source category
    into the per-genome counts table (v0.7.0)
  repo: https://github.com/wwood/CoverM
- name: fastp
  role: Quality-checks reads and removes leftover adapter sequences and low-quality
    reads before mapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:a9486fd1547e8b03c6ad80d3df5ed2cf1c599ee95a744227582ff8556a9a3fba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - SourceApp
  - bwa
  - bwa (mem)
  - fastp
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# fecal-source-tracking-read-mapping

## Summary

Maps shotgun metagenomic reads to source-associated reference genome databases with bwa mem and summarizes per-genome mapped read counts/depth with CoverM, the read-mapping core of SourceApp's metagenomic fecal source tracking (FST) framework.

## When to use

Use when assigning short-read shotgun metagenomic reads (e.g., 2 × 150-bp Illumina NovaSeq 6000) from waterway samples to curated genome databases with known source-associations to quantify fecal signal per source category.

## When NOT to use

- Input is long-read data or an already-assembled metagenome rather than short paired-end reads — SourceApp is designed to apportion fecal signal in short read metagenomes
- The goal is taxonomic profiling against a general database rather than attribution to genomes with known source-associations (e.g., fecal vs. lake water categories)
- Potential fecal sources must be distinguished solely from marker genes without reference genomes — this framework requires a genome-to-source-association database

## Inputs

- Gzipped paired-end shotgun metagenomic FASTQ reads (e.g., mesocosm 2 × 150-bp Illumina NovaSeq 6000 reads)
- Source-associated reference genome database (SourceApp default database from Zenodo 10.5281/zenodo.12795076 or user-built via sourceapp_build.py)
- Source-category annotations for each database genome

## Outputs

- Per-genome tab-separated counts table with mapped read counts and sequence depth for each database genome, grouped by source category
- Per-source attribution/apportionment estimates (cell-fraction-like metrics when --use-geq is used)

## How to apply

Download the mesocosm shotgun reads (NCBI BioProject PRJNA1161616) and the SourceApp default genome database (Zenodo 10.5281/zenodo.12795076). Quality-check and trim the paired-end FASTQs with fastp to remove leftover adapter sequences and low-quality reads, then index every genome in the source-associated database with bwa index. Map the trimmed reads to the database with bwa mem, applying SourceApp's optimal read-to-genome assignment thresholds: 93% minimum read-to-genome alignment identity (-r/--percent-identity 0.93), 70% minimum alignment-to-read overlap (-q/--query-coverage 0.7), and 0% target genome masking. Summarize per-genome sequence depth and mapped read counts grouped by source category using CoverM v0.7.0, writing a per-genome counts table; SourceApp then normalizes by genome equivalents (--use-geq) to approximate prokaryotic cell fractions and applies a 0.01% relative abundance limit of detection.

## Related tools

- **SourceApp** (Automates read mapping to source-associated genome databases, normalization by genome equivalents, and fecal source attribution/apportionment) — https://github.com/bglindner/SourceApp
- **bwa** (Indexes the genome database (bwa index) and maps trimmed paired-end reads to it with the mem algorithm)
- **CoverM** (Summarizes per-genome sequence depth and mapped read counts per source category into the per-genome counts table (v0.7.0)) — https://github.com/wwood/CoverM
- **fastp** (Quality-checks reads and removes leftover adapter sequences and low-quality reads before mapping)

## Examples

```
bwa index db/*.fna && bwa mem db/genomes.fna trimmed_R1.fq.gz trimmed_R2.fq.gz | coverm genome -r db/genomes.fna --methods relative_abundance count -1 trimmed_R1.fq.gz -2 trimmed_R2.fq.gz > per_genome_counts.tsv
```

## Evaluation signals

- The CoverM output table exists, is tab-separated, and contains one row per database genome
- The table reports mapped read counts and sequence depth values for each genome grouped by source category as computed by CoverM
- Read assignment uses the tuned thresholds (93% identity, 70% alignment-to-read overlap, 0% masking); with the 0.01% LOD these yield ~0.898 specificity and 0.902 sensitivity
- Every source genome is indexed (bwa index completes for all database FASTAs) and both mates of each trimmed paired-end FASTQ are consumed by bwa mem
- Estimated cell fractions for each source exceed the 0.01% relative abundance limit of detection before a source is called

## Limitations

- SourceApp's performance varied substantially by source — cows were underestimated due to undersampling of cow fecal microbiomes (false negatives across all samples)
- Portions reported by SourceApp do not represent a mass balance but estimate proportions of cells based on genome equivalents
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations; persistence of species-level marker genomes in the environment is largely unknown
- Substantial residuals remained after linear correction of cell fraction estimates (r2 = 0.64)
- SourceApp works best when users supply supplementary reference genomes to bolster the default database

## Evidence

- [abstract] created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases: "created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases"
- [results] 93% identity/70% overlap parameters: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] fastp quality trimming: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [methods] CoverM usage: "CoverM (v0.7.0; https://github.com/wwood/CoverM)"
- [readme] SourceApp input expectation: "SourceApp is designed to automate all tasks necessary to detect and quantify fecal signal in metagenomes collected from the water environment if users will simply provide gzipped copies of the raw"
