---
name: metagenomic-taxonomic-profiling
description: Use when shotgun metagenome FASTQ reads (e.g., mesocosm samples from
  PRJNA1161616 and fecal spike-in slurries from PRJNA1092107) must be converted into
  Kraken2 taxonomic profiles or kmtricks k-mer count tables to serve as input for
  FEAST, metaSourceTracker, or decOM source tracking comparisons.
when_to_use_negative:
- The downstream tool is SourceApp itself, which maps reads to source-associated genome
  databases rather than consuming taxonomic profiles or k-mer tables
- Input is already a Kraken2 profile or kmtricks k-mer count table and only the source
  tracking step (FEAST/metaSourceTracker/decOM) remains
- Data are 16S amplicon reads rather than shotgun metagenome FASTQ, which the benchmark
  workflow does not cover
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: Kraken2
  role: Generates taxonomic profiles (default settings) for mesocosm and fecal slurry
    metagenomes, as required by metaSourceTracker and FEAST
- name: kmtricks
  role: Generates k-mer count tables following the decOM documentation, for input
    to decOM
- name: fastp
  role: Quality-checks sequences by removing leftover adapter sequences and low-quality
    reads prior to profiling
- name: CoverM
  role: Read coverage and relative abundance calculator used in the genome-to-source
    mapping framework that the profile-based alternatives are compared against
  repo: github.com/wwood/CoverM
- name: SourceApp
  role: Benchmark source attribution/apportionment tool whose sensitivity/specificity
    (near 0.90) the profile-based alternatives are scored against
  repo: github.com/bglindner/SourceApp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:9f4e9629c5c8fe34b979bd89306c109fd905a5c7d3b8b56e68ba5a94d64ade22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CoverM
  - FEAST
  - Kraken2
  - Python
  - SourceApp
  - decOM
  - fastp
  - kmtricks
  - metaSourceTracker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metagenomic-taxonomic-profiling

## Summary

Generate taxonomic profiles (Kraken2) and k-mer count tables (kmtricks) from shotgun metagenome FASTQ reads so that alternative source tracking tools (FEAST, metaSourceTracker, decOM) can be run and compared against SourceApp on the same spike-in mesocosm benchmark data.

## When to use

Use when shotgun metagenome FASTQ reads (e.g., mesocosm samples from PRJNA1161616 and fecal spike-in slurries from PRJNA1092107) must be converted into Kraken2 taxonomic profiles or kmtricks k-mer count tables to serve as input for FEAST, metaSourceTracker, or decOM source tracking comparisons.

## When NOT to use

- The downstream tool is SourceApp itself, which maps reads to source-associated genome databases rather than consuming taxonomic profiles or k-mer tables
- Input is already a Kraken2 profile or kmtricks k-mer count table and only the source tracking step (FEAST/metaSourceTracker/decOM) remains
- Data are 16S amplicon reads rather than shotgun metagenome FASTQ, which the benchmark workflow does not cover

## Inputs

- Gzipped shotgun metagenome FASTQ reads (Illumina NovaSeq 6000 2x150-bp) from mesocosm samples (NCBI BioProject PRJNA1161616)
- Gzipped shotgun metagenome FASTQ reads from fecal spike-in slurries serving as source training data (NCBI BioProject PRJNA1092107)
- A Kraken2 database for taxonomic classification
- Ground truth mesocosm cell fractions (Supporting Information Table S1) for downstream scoring

## Outputs

- Kraken2 taxonomic profile tables for mesocosm and fecal slurry samples (inputs to FEAST and metaSourceTracker)
- kmtricks k-mer count tables for mesocosm and source samples (input to decOM)
- tool_comparison_table.csv tabulating sensitivity/specificity of each tool against ground truth cell fractions

## How to apply

Start from quality-controlled shotgun reads: trim adapters and low-quality bases with fastp, then normalize depth with BBtools bbnorm (target=30, min=5) as done for the mesocosm/slurry reads in the SourceApp study. For FEAST and metaSourceTracker, run Kraken2 with default settings on both the mesocosm metagenomes and the fecal slurry source metagenomes to produce taxonomic profiles; these tools require taxonomic profile tables for both sink and source samples. For decOM, generate k-mer count tables with kmtricks following the decOM documentation instead of Kraken2 profiles. Run each alternative tool on the mesocosm benchmark samples using the fecal slurry profiles as source training data, then score predictions against the ground truth cell fractions from mesocosm construction (Supporting Information Table S1) to compute sensitivity and specificity alongside SourceApp's tuned results (93% identity, 70% query coverage, 0.01% LOD).

## Related tools

- **Kraken2** (Generates taxonomic profiles (default settings) for mesocosm and fecal slurry metagenomes, as required by metaSourceTracker and FEAST)
- **kmtricks** (Generates k-mer count tables following the decOM documentation, for input to decOM)
- **fastp** (Quality-checks sequences by removing leftover adapter sequences and low-quality reads prior to profiling)
- **CoverM** (Read coverage and relative abundance calculator used in the genome-to-source mapping framework that the profile-based alternatives are compared against) — github.com/wwood/CoverM
- **SourceApp** (Benchmark source attribution/apportionment tool whose sensitivity/specificity (near 0.90) the profile-based alternatives are scored against) — github.com/bglindner/SourceApp

## Examples

```
kraken2 --db $KRAKEN2_DB --gzip-compressed --threads 8 --output mesocosm_kraken2.txt mesocosm_R1.fq.gz mesocosm_R2.fq.gz
```

## Evaluation signals

- Kraken2 profiles exist for both the mesocosm sink samples and the fecal slurry source samples, since FEAST and metaSourceTracker require both
- kmtricks k-mer count tables are produced per the decOM documentation rather than from Kraken2 output
- Reads were fastp-trimmed and bbnorm-normalized (target=30, min=5) before profiling, matching the study's read preprocessing
- Each tool's predictions reproduce the reported pattern: high sensitivity but very poor specificity (0.30 for FEAST, 0 for decOM and metaSourceTracker) versus SourceApp's 0.898 specificity and 0.902 sensitivity
- SourceApp apportionment correlation with ground truth is 0.86, exceeding FEAST (0.79), decOM (0.74), and metaSourceTracker (0.60) in the reproduced table

## Limitations

- The alternatives show very low specificity (0 for decOM and metaSourceTracker, 0.30 for FEAST), so profile-based tools over-attribute sources on this benchmark
- SourceApp's performance varies substantially by source, with cows underestimated due to under sampling of cow fecal microbiomes
- Benchmarking is limited to the spike-in mesocosm dataset; no prior work had benchmarked metagenomic FST against gold standard spike-in mesocosm data sets

## Evidence

- [other] Generate taxonomic profiles with Kraken2 using default settings for both the mesocosm and fecal slurry metagenomes, as required by metaSourceTracker and FEAST.: [evidence span withheld — anchor status: not_found]
- [other] kmtricks k-mer tables for decOM: [evidence span withheld — anchor status: not_found]
- [methods] fastp read quality filtering: "the sequences were quality checked using fastp to remove leftover adapter sequences and low-quality reads"
- [methods] bbnorm read normalization: [evidence span withheld — anchor status: not_found]
- [abstract] SourceApp maps reads to genome databases: "inferring source attribution and apportionment by mapping the metagenomic data to these genome databases"
