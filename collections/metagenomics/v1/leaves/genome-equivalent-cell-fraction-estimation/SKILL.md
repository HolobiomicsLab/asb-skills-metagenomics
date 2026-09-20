---
name: genome-equivalent-cell-fraction-estimation
description: Use when shotgun metagenomic reads from water samples must be converted
  into per-source cell-fraction estimates for fecal source attribution and apportionment,
  and genome-equivalent normalization is needed to compare summed sequence depth across
  source categories.
when_to_use_negative:
- When the goal is a mass-balance of source contributions — the portions reported
  by SourceApp do not represent a mass balance but proportions of cells based on genome
  equivalents.
- When contamination sources lack representative genomes in the database (e.g., cow
  feces was underestimated due to undersampling of cow fecal microbiomes) — cell-fraction
  estimates will be biased.
- When the input is targeted amplicon data or a pre-computed taxonomy feature table
  rather than shotgun FASTQ reads mapped to genome databases.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: SourceApp
  role: Pipeline that automates read mapping to the source-associated genome database,
    genome-equivalent normalization, attribution (LOD thresholding), and apportionment
    outputs
  repo: https://github.com/bglindner/SourceApp
- name: CoverM
  role: Estimates per-genome sequence depth and mapped-read counts from bwa mem alignments
    of mesocosm reads against database genomes
  repo: https://github.com/wwood/CoverM
- name: MicrobeCensus
  role: Estimates genome equivalents from the metagenome so summed per-source sequence
    depth can be normalized to approximate prokaryotic cell fractions
- name: fastp
  role: Quality checking and trimming of leftover adapter sequences and low-quality
    reads before mapping
- name: bwa
  role: Maps trimmed metagenomic reads to database genomes (bwa mem) prior to depth
    estimation
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:229b4971879fc198ec69f5b5508e8042d9fcb8011fd802a2184f66a989501e97
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# genome-equivalent-cell-fraction-estimation

## Summary

Estimate per-source prokaryotic cell fractions from shotgun metagenomes by mapping reads (bwa mem via CoverM) to source-associated genome databases, summing per-genome sequence depth per source category, and normalizing summed depth to genome equivalents with MicrobeCensus. This is the quantitative core of SourceApp's metagenomic fecal source tracking (FST) framework.

## When to use

Use when shotgun metagenomic reads from water samples must be converted into per-source cell-fraction estimates for fecal source attribution and apportionment, and genome-equivalent normalization is needed to compare summed sequence depth across source categories.

## When NOT to use

- When the goal is a mass-balance of source contributions — the portions reported by SourceApp do not represent a mass balance but proportions of cells based on genome equivalents.
- When contamination sources lack representative genomes in the database (e.g., cow feces was underestimated due to undersampling of cow fecal microbiomes) — cell-fraction estimates will be biased.
- When the input is targeted amplicon data or a pre-computed taxonomy feature table rather than shotgun FASTQ reads mapped to genome databases.

## Inputs

- Gzipped paired-end shotgun metagenomic FASTQ read pairs (e.g., Illumina NovaSeq 6000 2 × 150-bp reads; PRJNA1161616 mesocosm runs)
- SourceApp source-associated genome database (default database from Zenodo DOI 10.5281/zenodo.12795076, or a de novo database built with sourceapp_build.py)
- Ground-truth source compositions for validation (Supporting Information Table S1 cell fractions)

## Outputs

- Per-sample attribution table (presence/absence of each source category) as CSV
- Per-sample apportionment table (Portion per present source) as CSV
- Per-sample fraction table (cell fractions in c/c units with --use-geq) as CSV
- Aggregate sensitivity/specificity metrics vs. ground truth (e.g., sourceapp_benchmark_metrics.csv)

## How to apply

Quality-check and trim reads with fastp, then map reads to the source-associated genome database with bwa mem and estimate per-genome sequence depth and mapped-read counts with CoverM (v0.7.0). Sum sequence depth across all genomes (source-specific and cross-reactive) within each source category and divide by genome equivalents estimated by MicrobeCensus to obtain cell fraction_j = sum(X_i)/GEQ in c/c units (reported with --use-geq). Call a source 'present' only when at least one source-specific genome is detected AND its estimated cell fraction exceeds the limit of detection (0.01% relative abundance, min-frac 0.0001); the benchmarked mapping parameters are 93% minimum read-to-genome alignment identity and 70% minimum alignment-to-read overlap with 0% genome masking. Compute apportionment portions for each present source as its summed specific-plus-cross-reactive cell fraction divided by the sum of cell fractions of all present sources, then score attribution/apportionment against ground-truth spike-in compositions (Table S1) — tuned parameters yielded 0.902 sensitivity and 0.898 specificity across the 35 mesocosms.

## Related tools

- **SourceApp** (Pipeline that automates read mapping to the source-associated genome database, genome-equivalent normalization, attribution (LOD thresholding), and apportionment outputs) — https://github.com/bglindner/SourceApp
- **CoverM** (Estimates per-genome sequence depth and mapped-read counts from bwa mem alignments of mesocosm reads against database genomes) — https://github.com/wwood/CoverM
- **MicrobeCensus** (Estimates genome equivalents from the metagenome so summed per-source sequence depth can be normalized to approximate prokaryotic cell fractions)
- **fastp** (Quality checking and trimming of leftover adapter sequences and low-quality reads before mapping)
- **bwa** (Maps trimmed metagenomic reads to database genomes (bwa mem) prior to depth estimation)

## Examples

```
sourceapp.py -1 sample_R1.fq.gz -2 sample_R2.fq.gz -d default_db -r 0.93 -q 0.7 --min-frac 0.0001 --use-geq
```

## Evaluation signals

- Overall attribution sensitivity and specificity across the 35 mesocosms should be near 0.90 (reported tuned values: 0.902 sensitivity, 0.898 specificity).
- A source should only be called present when at least one source-specific genome is detected AND its estimated cell fraction exceeds the 0.01% LOD (min-frac 0.0001).
- Apportionment portions of all present sources in a sample must sum to 1 (each portion = source fraction / sum of present-source fractions).
- Fraction outputs should be in cell-fraction (c/c) units when --use-geq is supplied; without it, values are unnormalized depth sums.
- Cell-fraction estimates should correlate with ground-truth spike-in compositions (Table S1), though residuals remain after linear correction (best r2 = 0.64).

## Limitations

- Portions do not represent a mass balance; they estimate proportions of cells belonging to source categories based on genome equivalents and may vary from mass-based expectations.
- Performance varies substantially by source: cow contributions were underestimated (false negatives across all samples) due to undersampling of cow fecal microbiomes and poor understanding of the prokaryotic content of cow feces at the genomic level.
- Species-level marker genomes have largely unknown persistence in the environment, and SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations.
- Substantial residuals remain after linear correction of cell fraction estimates even with the best-performing parameters (r2 = 0.64).

## Evidence

- [intro] Objective: develop and test a new metagenomic FST framework based on read mapping of metagenomic data to source-associated genome databases and normalization by genome equivalents to approximate cell fractions.: "we theorized and tested a new framework for metagenomic FST based on read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units i.e.,"
- [methods] Limit of detection threshold on estimated cell fraction: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [results] Tuned parameters yielded near-0.90 sensitivity and specificity: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [results] Portions are cell-proportion estimates, not mass balance: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories"
- [readme] README: --use-geq reports prokaryotic cell fraction metrics: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
