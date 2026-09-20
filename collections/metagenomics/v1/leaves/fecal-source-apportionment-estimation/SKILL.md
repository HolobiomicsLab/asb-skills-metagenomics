---
name: fecal-source-apportionment-estimation
description: Use when gzipped shotgun metagenomic FASTQ reads (e.g., from impaired
  waterways) must be attributed to and apportioned among fecal source categories (human,
  sewage, septage, cow, dog, bird) using a curated source-associated genome database.
when_to_use_negative:
- When the goal is to infer the age of contamination or identify nonconservative (growth-capable)
  populations — SourceApp cannot do either
- When a strict mass balance of source contributions is required — SourceApp portions
  are genome-equivalent-based cell proportion estimates, not mass-balance results
- When the source category of interest is poorly represented at the genomic level
  (e.g., cow feces) — false negatives were observed across all samples for cows due
  to under-sampling of cow fecal microbiomes
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: 'End-to-end pipeline that automates detection and quantification of fecal
    signal: read mapping to source genomes, attribution, apportionment, and CSV output'
  repo: github.com/bglindner/SourceApp
- name: MicrobeCensus
  role: Estimates genome equivalents used to normalize summed mapped sequence depths
    into approximate prokaryotic cell fractions (--use-geq mode)
- name: CoverM
  role: Read coverage/relative abundance calculator used for mapping metagenomic reads
    to genome databases in the framework (v0.7.0)
  repo: github.com/wwood/CoverM
- name: Python
  role: Implementation language; SourceApp requires Unix or Unix-like operating systems
    and is installed via conda/mamba
  repo: github.com/bglindner/SourceApp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:97fe2a7e3c864bbe513740613114b2dafd7b321efb44864201bcfc79a849957a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fecal-source-apportionment-estimation

## Summary

A metagenomic fecal source tracking (FST) framework, implemented in SourceApp, that infers which fecal sources are present and apportioned in water metagenomes by mapping shotgun reads to source-associated genome databases and normalizing mapped depths by genome equivalents to approximate prokaryotic cell fractions.

## When to use

Use when gzipped shotgun metagenomic FASTQ reads (e.g., from impaired waterways) must be attributed to and apportioned among fecal source categories (human, sewage, septage, cow, dog, bird) using a curated source-associated genome database.

## When NOT to use

- When the goal is to infer the age of contamination or identify nonconservative (growth-capable) populations — SourceApp cannot do either
- When a strict mass balance of source contributions is required — SourceApp portions are genome-equivalent-based cell proportion estimates, not mass-balance results
- When the source category of interest is poorly represented at the genomic level (e.g., cow feces) — false negatives were observed across all samples for cows due to under-sampling of cow fecal microbiomes

## Inputs

- Gzipped raw shotgun metagenomic reads in FASTQ format (short-read, e.g., Illumina NovaSeq 6000 2 x 150-bp)
- Source-associated reference genome database with known source associations (default database available on Zenodo, formatted for immediate use)
- Optional supplementary reference genomes to bolster the default database, or a de novo database built with sourceapp_build.py

## Outputs

- Attribution table (CSV): presence/absence call per fecal source category per metagenome
- Apportionment table (CSV): Portion of each present source (summed specific-plus-cross-reactive cell fraction divided by sum over present sources)
- Fraction table (CSV): Fraction in c/c units when --use-geq is called (metrics akin to prokaryotic cell fraction)

## How to apply

Map the metagenomic reads to the source-associated genome database (default database from Zenodo, doi 10.5281/zenodo.12795076; or a custom one built with sourceapp_build.py) using SourceApp's tuned parameters: 93% minimum read-to-genome alignment identity (-r 0.93), 70% minimum alignment-to-read overlap (-q 0.7), 0% target genome masking, and a limit of detection of 0.01% relative abundance. Sum per-genome sequence depths across all genomes (source-specific plus cross-reactive, flagged at ANI >= 95% between source categories) in each source category and normalize by genome equivalents (estimated with MicrobeCensus via --use-geq) to obtain cell fraction_j = sum(X_i)/GEQ. Call a source present only when at least one source-specific genome is detected (sequence breadth above the -l/--limit-threshold) AND its estimated cell fraction exceeds the 0.01% LOD; compute apportionment portions as each present source's summed cell fraction divided by the sum over all present sources. Write the attribution (presence/absence), apportionment (Portion), and fractioning (Fraction, c/c units) tables as CSV outputs. Validate against ground-truth cell fractions when available; the tuned parameters gave specificity 0.898 and sensitivity 0.902 in spike-in mesocosm benchmarks.

## Related tools

- **SourceApp** (End-to-end pipeline that automates detection and quantification of fecal signal: read mapping to source genomes, attribution, apportionment, and CSV output) — github.com/bglindner/SourceApp
- **MicrobeCensus** (Estimates genome equivalents used to normalize summed mapped sequence depths into approximate prokaryotic cell fractions (--use-geq mode))
- **CoverM** (Read coverage/relative abundance calculator used for mapping metagenomic reads to genome databases in the framework (v0.7.0)) — github.com/wwood/CoverM
- **Python** (Implementation language; SourceApp requires Unix or Unix-like operating systems and is installed via conda/mamba) — github.com/bglindner/SourceApp

## Examples

```
python sourceapp.py -1 mesocosm_R1.fastq.gz -2 mesocosm_R2.fastq.gz --use-geq -r 0.93 -q 0.7
```

## Evidence

- [abstract] created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases: "created an associated bioinformatic tool, called SourceApp, for inferring source attribution and apportionment by mapping the metagenomic data to these genome databases"
- [results] tuned parameters yielding 0.898 specificity and 0.902 sensitivity: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] limit of detection rule for source presence: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [results] portions are not a mass balance: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells"
- [readme] --use-geq reports cell-fraction-like metrics: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called)"
- [readme] supplementary genomes improve performance: "SourceApp works best when the user is able to provide supplementary reference genomes to bolster SourceApp's default genomic reference database."
