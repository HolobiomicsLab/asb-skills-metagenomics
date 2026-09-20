---
name: metagenomic-read-alignment-to-genome-databases
description: Use when assigning shotgun metagenomic reads from water or mesocosm samples
  (e.g. FASTQ reads from PRJNA1161616) to individual genomes in a source-associated
  database to quantify per-genome and per-source-category signal.
when_to_use_negative:
- The analysis goal is taxonomic profiling against a general reference catalog rather
  than attribution to source-associated genomes with known source labels
- Input data are long Nanopore reads rather than short paired-end reads; bwa mem short-read
  mapping with the 93% identity / 70% overlap thresholds was tuned for 2 × 150-bp
  short-read metagenomes
- You need a mass balance of contaminant mass rather than cell-fraction estimates;
  SourceApp portions are based on genome equivalents and do not represent a mass balance
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: SourceApp
  role: Pipeline that automates read mapping to the source-associated database and
    applies the 93% identity / 70% query coverage / 0% masking thresholds for source
    attribution
  repo: https://github.com/bglindner/SourceApp
- name: bwa
  role: Indexes database genomes (bwa index) and aligns trimmed paired-end reads with
    the bwa mem algorithm
- name: CoverM
  role: Summarizes per-genome sequence depth and mapped read counts per source category
    (v0.7.0), producing the per-genome counts table
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
  source_content_hash: sha256:01778a833ce559ebf2f9c2c542f6ae913bdbc722e72204f1ea0dd1a4f6e3e7bf
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metagenomic-read-alignment-to-genome-databases

## Summary

Maps quality-trimmed shotgun metagenomic reads to a curated source-associated reference genome database with bwa mem and summarizes per-genome mapped read counts and sequence depth with CoverM, enabling read-mapping-based fecal source tracking as implemented in SourceApp.

## When to use

Use when assigning shotgun metagenomic reads from water or mesocosm samples (e.g. FASTQ reads from PRJNA1161616) to individual genomes in a source-associated database to quantify per-genome and per-source-category signal.

## When NOT to use

- The analysis goal is taxonomic profiling against a general reference catalog rather than attribution to source-associated genomes with known source labels
- Input data are long Nanopore reads rather than short paired-end reads; bwa mem short-read mapping with the 93% identity / 70% overlap thresholds was tuned for 2 × 150-bp short-read metagenomes
- You need a mass balance of contaminant mass rather than cell-fraction estimates; SourceApp portions are based on genome equivalents and do not represent a mass balance

## Inputs

- Gzipped paired-end shotgun metagenomic FASTQ reads (e.g. mesocosm reads from NCBI BioProject PRJNA1161616)
- Source-associated genome database in FASTA format (SourceApp default database from Zenodo 10.5281/zenodo.12795076 or a user-built sourceapp_build.py database)
- bwa index files generated from the database FASTA

## Outputs

- BAM alignments of metagenomic reads against the genome database (bwa mem)
- Tab-separated per-genome counts table from CoverM v0.7.0 reporting mapped read counts and sequence depth for each database genome, grouped by source category

## How to apply

Download the mesocosm shotgun reads (2 × 150-bp from an Illumina NovaSeq 6000, BioProject PRJNA1161616) and the SourceApp default genome database from Zenodo (10.5281/zenodo.12795076). Quality-check and trim the paired-end FASTQ reads with fastp to remove leftover adapter sequences and low-quality reads, then index every genome in the database with bwa index. Align the trimmed reads to the database with the bwa mem algorithm, then summarize per-genome sequence depth and mapped read counts per source category using CoverM v0.7.0 to produce a per-genome counts table. Downstream source assignment relies on SourceApp's tuned read-to-genome thresholds: 93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap (query coverage), and 0% target genome masking, with a limit of detection of 0.01% relative abundance. Rationale: mapping reads to genome equivalents and normalizing by genome size approximates prokaryotic cell fractions, which SourceApp converts into source attribution and apportionment.

## Related tools

- **SourceApp** (Pipeline that automates read mapping to the source-associated database and applies the 93% identity / 70% query coverage / 0% masking thresholds for source attribution) — https://github.com/bglindner/SourceApp
- **bwa** (Indexes database genomes (bwa index) and aligns trimmed paired-end reads with the bwa mem algorithm)
- **CoverM** (Summarizes per-genome sequence depth and mapped read counts per source category (v0.7.0), producing the per-genome counts table) — https://github.com/wwood/CoverM
- **fastp** (Quality-checks reads and removes leftover adapter sequences and low-quality reads before mapping)

## Examples

```
coverm genome --mapper bwa-mem -r sourceapp_database.fna -1 mesocosm_R1.trimmed.fq.gz -2 mesocosm_R2.trimmed.fq.gz --methods count
```

## Evaluation signals

- The CoverM output table exists and is tab-separated with exactly one row per database genome
- The table reports mapped read counts and sequence depth values for each genome, grouped by source category as computed by CoverM
- Read-to-genome assignments use the tuned thresholds: 93% minimum alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking
- Per-source estimated cell fractions fall below the 0.01% limit of detection are treated as undetected
- Sensitivity and specificity of the resulting source attribution against the spike-in mesocosm ground truth are near 0.90 (0.902 sensitivity, 0.898 specificity at optimal parameters)

## Limitations

- SourceApp's performance varied substantially by source; cows were underestimated due to undersampling of cow fecal microbiomes, giving false negatives across all samples
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations, and persistence of species-level marker genomes in the environment is largely unknown
- Portions reported are estimates of cell proportions based on genome equivalents and do not represent a mass balance; substantial residuals remained after linear correction (r2 = 0.64)
- The default database works best when the user supplies supplementary reference genomes; minimum alignment-to-read overlap was the least impactful parameter (top sets spanned 30–90%)

## Evidence

- [results] Optimal parameters: 93% identity, 70% overlap, 0% masking: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] fastp trimming step: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [methods] CoverM version and role: "CoverM (v0.7.0; https://github.com/wwood/CoverM)"
- [abstract] SourceApp framework: map metagenomic data to genome databases: "inferring source attribution and apportionment by mapping the metagenomic data to these genome databases"
- [readme] README: SourceApp requires gzipped FASTQ reads and a source-associated database: "SourceApp requires a database of reference genomes with known source-associations."
