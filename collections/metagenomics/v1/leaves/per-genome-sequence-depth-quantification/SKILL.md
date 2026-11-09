---
name: per-genome-sequence-depth-quantification
description: Use when short-read metagenomic FASTQ data must be quantified against
  a set of reference genomes to obtain per-genome mapped read counts and sequence
  depth, e.g. for read-mapping-based source attribution.
when_to_use_negative:
- Input is already a per-genome abundance or coverage table — re-running the mapping
  adds no information
- Long-read metagenomic data requiring different filtering/alignment (the SourceApp
  framework and these thresholds were tuned for short Illumina reads; long reads were
  only used to recover MAGs via filtlong-filtered minION data)
- The goal is de novo source database construction rather than quantifying reads against
  an existing curated database (use sourceapp_build.py instead)
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: bwa (mem)
  role: Maps trimmed paired-end metagenomic reads to all bwa-indexed genomes in the
    source-associated database
- name: CoverM
  role: v0.7.0; summarizes per-genome sequence depth and number of reads mapped to
    genomes in each source category, writing the per-genome counts table
  repo: https://github.com/wwood/CoverM
- name: fastp
  role: Quality-checks reads and removes leftover adapter sequences and low-quality
    reads before mapping
- name: SourceApp
  role: Wraps the mapping-and-quantification workflow and assigns reads to source-associated
    genomes using the 93% identity / 70% overlap alignment thresholds
  repo: https://github.com/bglindner/SourceApp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:6574774951161b444c7e98e22caaa5280808b0a6d401757191b9e5dcd4fa56fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wwood/CoverM
  tools:
  - CoverM
  - SourceApp
  - bwa (mem)
  - fastp
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Per-genome sequence depth quantification

## Summary

Summarizes how many metagenomic reads map to each genome in a source-associated reference database, and at what sequence depth, using CoverM v0.7.0 after bwa mem mapping — the read quantification backbone of SourceApp's fecal source tracking.

## When to use

Use when short-read metagenomic FASTQ data must be quantified against a set of reference genomes to obtain per-genome mapped read counts and sequence depth, e.g. for read-mapping-based source attribution.

## When NOT to use

- Input is already a per-genome abundance or coverage table — re-running the mapping adds no information
- Long-read metagenomic data requiring different filtering/alignment (the SourceApp framework and these thresholds were tuned for short Illumina reads; long reads were only used to recover MAGs via filtlong-filtered minION data)
- The goal is de novo source database construction rather than quantifying reads against an existing curated database (use sourceapp_build.py instead)

## Inputs

- Gzipped paired-end shotgun metagenomic reads in FASTQ format (e.g. mesocosm reads from NCBI BioProject PRJNA1161616, 2 × 150-bp Illumina NovaSeq 6000)
- Source-associated genome database FASTA files (SourceApp default database from Zenodo 10.5281/zenodo.12795076) with source-category annotations
- bwa index files for the database genomes

## Outputs

- Tab-separated per-genome counts table with one row per database genome, reporting mapped read counts and sequence depth values grouped by source category as computed by CoverM v0.7.0

## How to apply

Download the mesocosm shotgun reads (NCBI BioProject PRJNA1161616) and the SourceApp default genome database from Zenodo (10.5281/zenodo.12795076). Quality-check and trim the paired-end reads with fastp to remove leftover adapter sequences and low-quality reads. Index every genome in the source-associated database with bwa index, then map the trimmed reads with bwa mem. Summarize per-genome sequence depth and mapped read counts, grouped by source category, with CoverM v0.7.0, writing the per-genome counts table. SourceApp's read-to-genome assignment applies alignment thresholds of 93% minimum read-to-genome alignment identity (-r/--percent-identity default 0.93), 70% minimum alignment-to-read overlap (-q/--query-coverage default 0.7), and 0% target genome masking, with a sequence breadth detection threshold (-l/--limit-threshold) and a 0.01% relative-abundance limit of detection.

## Related tools

- **bwa (mem)** (Maps trimmed paired-end metagenomic reads to all bwa-indexed genomes in the source-associated database)
- **CoverM** (v0.7.0; summarizes per-genome sequence depth and number of reads mapped to genomes in each source category, writing the per-genome counts table) — https://github.com/wwood/CoverM
- **fastp** (Quality-checks reads and removes leftover adapter sequences and low-quality reads before mapping)
- **SourceApp** (Wraps the mapping-and-quantification workflow and assigns reads to source-associated genomes using the 93% identity / 70% overlap alignment thresholds) — https://github.com/bglindner/SourceApp

## Examples

```
bwa mem reference_genomes/combined_genomes.fna trimmed_R1.fq.gz trimmed_R2.fq.gz | coverm genome -r combined_genomes.fna --methods count mean --min-covered-fraction 0.7 -1 trimmed_R1.fq.gz -2 trimmed_R2.fq.gz
```

## Evaluation signals

- The per-genome counts table exists and is tab-separated with exactly one row per database genome
- The table reports both mapped read counts and sequence depth values for each genome, grouped by source category as computed by CoverM
- Reads aligning below 93% identity or with less than 70% of the read covered by the alignment are not attributed to the genome
- Sources whose estimated cell fraction falls below the 0.01% relative-abundance limit of detection are reported as absent
- Mapping success can be cross-checked by confirming reads were fastp-trimmed and that the reference FASTAs were indexed with bwa index before bwa mem

## Limitations

- SourceApp portions do not represent a mass balance but estimate proportions of cells belonging to source categories based on genome equivalents, so they may vary from mass-based expectations
- SourceApp works best when users provide supplementary reference genomes to bolster the default database; cow sources were underestimated due to under sampling of cow fecal microbiomes, with false negatives for cows across all samples
- Persistence of the species-level marker genomes in the environment is largely unknown, and the tool cannot infer the age of contamination or identify nonconservative (growth-capable) populations

## Evidence

- [methods] Summarize per-genome sequence depth and number of reads mapped to genomes in each source category using CoverM v0.7.0: [evidence span withheld — anchor status: not_found]
- [results] optimal read-to-genome assignment parameters: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] fastp quality trimming of reads: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [readme] CoverM description from README: [evidence span withheld — anchor status: external_doc]
- [methods] percent identity filter default: "Minimum BLAST-like percent identity of alignment between read and reference genome (float; default 0.93)"
