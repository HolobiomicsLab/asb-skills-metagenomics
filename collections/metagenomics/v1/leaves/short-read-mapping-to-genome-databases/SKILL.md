---
name: short-read-mapping-to-genome-databases
description: Use when paired-end shotgun metagenomic FASTQ reads from water samples
  must be attributed to competing fecal source categories (e.g., human, sewage, septage,
  cow, dog, goose) using a genome-to-source database without sequencing the sources
  themselves.
when_to_use_negative:
- When potential fecal sources are available and cheap to sequence directly, and a
  de novo source database is preferred over the default genome database
- When reads are long-read (Oxford Nanopore) rather than short reads, since the pipeline
  and benchmark are built around short-read mapping with bwa mem
- When a mass-balance answer is required, since SourceApp portions estimate proportions
  of cells by genome equivalents and do not represent a mass balance
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SourceApp
  role: Main Python pipeline that automates read QC, mapping, genome-equivalent normalization,
    source detection (LOD filtering), and apportionment
  repo: https://github.com/bglindner/SourceApp
- name: fastp
  role: Quality checking and trimming of leftover adapter sequences and low-quality
    reads prior to mapping
- name: bwa
  role: bwa mem alignment of trimmed reads to database reference genomes
- name: CoverM
  role: Estimates per-genome sequence depth and mapped-read counts from alignments
    to compute genome coverage
  repo: https://github.com/wwood/CoverM
- name: MicrobeCensus
  role: Estimates average genome size per sample so summed sequence depth can be normalized
    to genome equivalents (cell fractions)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:f12ae025b7e13d3a9cd5245df5fb79d15de3f5e763a984b2244b9e80db34f7d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  - bwa
  - fastp
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# short-read-mapping-to-genome-databases

## Summary

A metagenomic fecal source tracking method (SourceApp) that maps short shotgun reads to curated source-associated genome databases and normalizes mapped sequence depth to genome-equivalent cell fractions to infer source attribution and apportionment.

## When to use

Use when paired-end shotgun metagenomic FASTQ reads from water samples must be attributed to competing fecal source categories (e.g., human, sewage, septage, cow, dog, goose) using a genome-to-source database without sequencing the sources themselves.

## When NOT to use

- When potential fecal sources are available and cheap to sequence directly, and a de novo source database is preferred over the default genome database
- When reads are long-read (Oxford Nanopore) rather than short reads, since the pipeline and benchmark are built around short-read mapping with bwa mem
- When a mass-balance answer is required, since SourceApp portions estimate proportions of cells by genome equivalents and do not represent a mass balance

## Inputs

- Gzipped paired-end short-read shotgun metagenomic FASTQ files (e.g., Illumina NovaSeq 6000 2 x 150-bp reads from freshwater mesocosms, SRA BioProject PRJNA1161616)
- Source-associated genome database with known source-associations (SourceApp default database from Zenodo doi:10.5281/zenodo.12795076, or a de novo database built with sourceapp_build.py)

## Outputs

- Per-sample source attribution table (which source categories are present)
- Per-sample source apportionment table (normalized fractions summing over present sources)
- Per-sample estimated genome-equivalent cell fraction table
- Aggregate sensitivity and specificity metrics (e.g., sourceapp_benchmark_metrics.csv)

## How to apply

Run sourceapp.py on each gzipped paired-end read pair against the SourceApp default database (Zenodo 10.5281/zenodo.12795076) using the tuned parameters: percent identity 0.93 (-r), query coverage 0.7 (-q), limit of detection 0.01% (min-frac 0.0001), and --use-geq to report genome-equivalent cell fractions. Internally, reads are quality-checked and trimmed with fastp, mapped to database genomes with bwa mem, and per-genome sequence depth and mapped-read counts are estimated with CoverM (v0.7.0). Summed sequence depth per source category is normalized to genome equivalents via MicrobeCensus to estimate prokaryotic cell fractions. A source is called 'present' only if at least one source-specific genome is detected and its estimated cell fraction exceeds the 0.01% LOD; apportionment is computed by normalizing each present source's fraction (specific plus cross-reactive) by the sum over all present sources. This design allows analysis without sequencing potential fecal sources because reads are assigned via curated genome-to-source associations rather than de novo source profiling.

## Related tools

- **SourceApp** (Main Python pipeline that automates read QC, mapping, genome-equivalent normalization, source detection (LOD filtering), and apportionment) — https://github.com/bglindner/SourceApp
- **fastp** (Quality checking and trimming of leftover adapter sequences and low-quality reads prior to mapping)
- **bwa** (bwa mem alignment of trimmed reads to database reference genomes)
- **CoverM** (Estimates per-genome sequence depth and mapped-read counts from alignments to compute genome coverage) — https://github.com/wwood/CoverM
- **MicrobeCensus** (Estimates average genome size per sample so summed sequence depth can be normalized to genome equivalents (cell fractions))

## Examples

```
sourceapp.py -1 mesocosm_R1.fastq.gz -2 mesocosm_R2.fastq.gz -d default_db -r 0.93 -q 0.7 --min-frac 0.0001 --use-geq
```

## Evaluation signals

- On the fecal spike-in freshwater mesocosm benchmark (n = 31 metagenomes plus 1 negative control), overall sensitivity should be near 0.902 and specificity near 0.898 when run with the tuned parameter set
- A source is only called present if at least one source-specific genome is detected and its estimated cell fraction exceeds the 0.01% LOD (min-frac 0.0001)
- Apportionment fractions across present sources must be normalized (each source's specific + cross-reactive fraction divided by the sum over all present sources), so portions sum to 1
- Alignment statistics should reflect the tuned filters: minimum 93% read-to-genome identity and 70% of the read covered by the alignment
- Per-sample attribution, apportionment, and fraction tables should be emitted as CSVs that can be scored against ground-truth spike-in compositions (Table S1)

## Limitations

- Performance varied substantially by source; cows were underestimated/missed across samples, attributed to undersampling of cow fecal microbiomes and poor understanding of the prokaryotic content of cow feces at the genomic level
- The portions reported by SourceApp do not represent a mass balance but rather estimates of the proportions of cells belonging to source categories based on genome equivalents
- The species-level representative genomes used as markers represent populations with largely unknown persistence in the environment, and SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations
- SourceApp works best when the user supplies supplementary reference genomes to bolster the default genomic reference database

## Evidence

- [abstract] created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases: "created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases"
- [results] Optimal parameters yielded 0.898 specificity and 0.902 sensitivity: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] Percent identity filter definition: "Minimum BLAST-like percent identity of alignment between read and reference genome (float; default 0.93)"
- [methods] LOD detection rule: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [methods] fastp QC step: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [readme] Genome-equivalent reporting in README: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called)"
