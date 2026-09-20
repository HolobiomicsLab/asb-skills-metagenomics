---
name: metagenomic-fecal-source-tracking
description: Use when gzipped FASTQ shotgun metagenomes from water samples must be
  attributed to specific fecal source categories and apportioned by cell fraction,
  without sequencing the potential sources themselves.
when_to_use_negative:
- The input is amplicon (16S) data or long-read metagenomes rather than short-read
  shotgun data — SourceApp is a short-read (bwa mem) mapping workflow
- Potential fecal sources are available for direct sequencing and a de novo site-specific
  database is preferred — build one with sourceapp_build.py instead of relying solely
  on the default database
- The goal is a mass balance of contamination or dating contamination events — SourceApp
  portions do not represent a mass balance and the tool cannot infer contamination
  age
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: End-to-end pipeline that performs read mapping, genome-equivalent normalization,
    source attribution, and apportionment from raw FASTQ inputs
  repo: https://github.com/bglindner/SourceApp
- name: CoverM
  role: Estimates per-genome sequence depth and mapped-read counts after bwa mem read
    mapping within the SourceApp pipeline
  repo: https://github.com/wwood/CoverM
- name: fastp
  role: Quality-checks and trims reads (adapter and low-quality read removal) before
    mapping inside SourceApp
- name: bwa
  role: Maps quality-trimmed reads to the source-associated genome database genomes
- name: MicrobeCensus
  role: pip-installed dependency used to normalize summed sequence depth per source
    category to genome-equivalent cell fractions
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:3cecd5e5c7b95378d22d88e89af7d14b051c13492ef6fa2e6d4b54e6441c1b1c
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metagenomic-fecal-source-tracking

## Summary

A metagenomic fecal source tracking (FST) skill that attributes and apportions fecal contamination amongst competing source categories (cat, chicken, cow, dog, goat, pig, sewage, septage) by mapping shotgun metagenomic reads to source-associated genome databases and normalizing to genome-equivalent cell fractions using SourceApp.

## When to use

Use when gzipped FASTQ shotgun metagenomes from water samples must be attributed to specific fecal source categories and apportioned by cell fraction, without sequencing the potential sources themselves.

## When NOT to use

- The input is amplicon (16S) data or long-read metagenomes rather than short-read shotgun data — SourceApp is a short-read (bwa mem) mapping workflow
- Potential fecal sources are available for direct sequencing and a de novo site-specific database is preferred — build one with sourceapp_build.py instead of relying solely on the default database
- The goal is a mass balance of contamination or dating contamination events — SourceApp portions do not represent a mass balance and the tool cannot infer contamination age

## Inputs

- Gzipped paired-end shotgun metagenomic FASTQ files (e.g. Illumina NovaSeq 6000 2 x 150-bp reads)
- SourceApp default database of source-associated reference genomes and MAGs (Zenodo 10.5281/zenodo.12795076)
- Optional ground-truth source compositions (e.g. Table S1 spike-in cellular fractions) for benchmarking

## Outputs

- Per-sample source attribution table (which source categories are present)
- Per-sample source apportionment table (normalized fractions per present source)
- Per-genome estimated cell fraction table
- Aggregate sensitivity and specificity metrics (e.g. sourceapp_benchmark_metrics.csv)

## How to apply

Run sourceapp.py on each paired-end read set against the SourceApp default database (Zenodo 10.5281/zenodo.12795076) with the tuned parameters: percent identity (-r) 0.93, query coverage (-q) 0.7, 0% target genome masking, and a limit of detection of 0.01% relative abundance (min-frac 0.0001). Internally the tool quality-checks and trims reads with fastp, maps reads to database genomes with bwa mem, and estimates per-genome sequence depth and mapped-read counts with CoverM (v0.7.0). Call --use-geq so that summed sequence depth per source category is normalized to genome equivalents via MicrobeCensus, approximating prokaryotic cell fractions. Call a source 'present' only if at least one source-specific genome is detected (sequence breadth threshold) AND its estimated cell fraction exceeds the 0.01% LOD, then apportion by normalizing each present source's fraction (specific plus cross-reactive genomes) by the sum over all present sources. Score attributions against known spike-in or ground-truth compositions to compute sensitivity and specificity; the tuned parameters yielded 0.898 specificity and 0.902 sensitivity on the 35 freshwater mesocosms.

## Related tools

- **SourceApp** (End-to-end pipeline that performs read mapping, genome-equivalent normalization, source attribution, and apportionment from raw FASTQ inputs) — https://github.com/bglindner/SourceApp
- **CoverM** (Estimates per-genome sequence depth and mapped-read counts after bwa mem read mapping within the SourceApp pipeline) — https://github.com/wwood/CoverM
- **fastp** (Quality-checks and trims reads (adapter and low-quality read removal) before mapping inside SourceApp)
- **bwa** (Maps quality-trimmed reads to the source-associated genome database genomes)
- **MicrobeCensus** (pip-installed dependency used to normalize summed sequence depth per source category to genome-equivalent cell fractions)

## Examples

```
sourceapp.py -1 sample_R1.fq.gz -2 sample_R2.fq.gz -r 0.93 -q 0.7 --min-frac 0.0001 --use-geq -d sourceapp_default_db
```

## Evaluation signals

- Overall benchmark sensitivity and specificity near 0.90 (paper's tuned values: 0.902 sensitivity, 0.898 specificity) when scored against known spike-in mesocosm compositions
- A source is called present only when at least one source-specific genome is detected AND its estimated cell fraction exceeds the 0.01% LOD — check no attribution violates this rule
- Per-source attribution table contains 8 source rows (cat, chicken, cow, dog, goat, pig, sewage, septage) with sensitivity/specificity values in [0, 1]
- Apportionment fractions sum to 1 across present sources after normalization of specific plus cross-reactive genome fractions
- Attribution CSVs, apportionment CSVs, and fraction tables are written for every sample (n = 35 mesocosms including n = 1 negative control)

## Limitations

- Cow sources were reported as false negatives across all mesocosms containing cow spike-in signal, due to under sampling of the cow fecal microbiome (759 species in the goat database vs 43 in the cow database)
- SourceApp portions do not represent a mass balance but rather estimates of the proportions of cells belonging to source categories based on genome equivalents
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations; persistence of species-level marker genomes in the environment is largely unknown
- Substantial residuals remained after linear correction of cell fraction estimates even with the best performing parameters (r2 = 0.64)
- SourceApp works best when users provide supplementary reference genomes to bolster the default database

## Evidence

- [results] Optimal parameters yielded 0.898 specificity and 0.902 sensitivity: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898"
- [abstract] Cow sources underestimated due to database under sampling: "SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes."
- [intro] Genome-equivalent normalization rationale: "we theorized and tested a new framework for metagenomic FST based on read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units i.e.,"
- [methods] fastp quality filtering inside the pipeline: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [methods] Percent identity filter default: "Minimum BLAST-like percent identity of alignment between read and reference genome (float; default 0.93)"
- [readme] Tool usage and --use-geq flag: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
- [readme] Bolstering the default database improves results: "SourceApp works best when the user is able to provide supplementary reference genomes to bolster SourceApp's default genomic reference database."
