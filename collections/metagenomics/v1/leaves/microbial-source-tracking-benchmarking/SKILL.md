---
name: microbial-source-tracking-benchmarking
description: Use when comparing fecal source tracking tools on shotgun metagenomes
  with ground-truth source compositions (e.g., spike-in mesocosms), or when deciding
  which FST tool and tuning parameters to adopt.
when_to_use_negative:
- Input data are 16S amplicon reads or otherwise not short-read shotgun metagenomes
  in FASTQ format, since SourceApp and this benchmark are built on shotgun data
- The goal is taxonomic profiling, strain-level outbreak tracking, or dating contamination
  rather than fecal source attribution/apportionment — SourceApp cannot infer contamination
  age or identify nonconservative (growth-capable) populations
- Expected sources include categories with poorly represented reference genomes (e.g.,
  cow feces), where apportionment estimates are known to be unreliable (cows were
  underestimated and showed false negatives)
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SourceApp
  role: Benchmarked tool under evaluation; performs source attribution and apportionment
    by mapping reads to source-associated genome databases at tuned parameters (93%
    identity, 70% overlap, 0.01% LOD)
  repo: github.com/bglindner/SourceApp
- name: FEAST
  role: Alternative source tracking tool benchmarked on Kraken2 profiles of mesocosm
    and fecal slurry data
- name: metaSourceTracker
  role: Alternative source tracking tool benchmarked on Kraken2 profiles of mesocosm
    and fecal slurry data
- name: decOM
  role: Alternative k-mer-based source tracking tool benchmarked on kmtricks k-mer
    count tables
- name: Kraken2
  role: Generates the taxonomic profiles required by FEAST and metaSourceTracker
- name: kmtricks
  role: Generates k-mer count tables required as decOM input
- name: CoverM
  role: Read coverage/relative abundance calculator cited among the article's methods
    resources
  repo: github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:26dba28a7a237259c7590d72d3e9434e305d534ea07af28952fbd0e118725167
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
  - kmtricks
  - metaSourceTracker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# microbial-source-tracking-benchmarking

## Summary

A benchmarking skill for comparing metagenomic fecal source tracking and apportionment tools (SourceApp, FEAST, metaSourceTracker, decOM) on spike-in mesocosm metagenomes with known ground-truth cell fractions, using sensitivity/specificity and correlation with true source proportions as the metrics.

## When to use

Use when comparing fecal source tracking tools on shotgun metagenomes with ground-truth source compositions (e.g., spike-in mesocosms), or when deciding which FST tool and tuning parameters to adopt.

## When NOT to use

- Input data are 16S amplicon reads or otherwise not short-read shotgun metagenomes in FASTQ format, since SourceApp and this benchmark are built on shotgun data
- The goal is taxonomic profiling, strain-level outbreak tracking, or dating contamination rather than fecal source attribution/apportionment — SourceApp cannot infer contamination age or identify nonconservative (growth-capable) populations
- Expected sources include categories with poorly represented reference genomes (e.g., cow feces), where apportionment estimates are known to be unreliable (cows were underestimated and showed false negatives)

## Inputs

- Gzipped shotgun metagenomic FASTQ reads of mesocosm samples (BioProject PRJNA1161616)
- Fecal spike-in slurry reads as source training data (BioProject PRJNA1092107)
- Kraken2 taxonomic profiles for FEAST and metaSourceTracker
- kmtricks k-mer count tables for decOM
- SourceApp database of reference genomes with known source associations (Zenodo 10.5281/zenodo.12795076)
- Ground-truth cell fractions from mesocosm construction (Supporting Information Table S1)

## Outputs

- tool_comparison_table.csv with sensitivity/specificity (and correlation with ground truth) for each tool
- Per-sample source attribution and apportionment estimates (cell-fraction-like portions) from each tool

## How to apply

Download the shotgun mesocosm reads from NCBI BioProject PRJNA1161616 and the fecal spike-in slurry reads from PRJNA1092107. Generate Kraken2 taxonomic profiles (default settings) for both mesocosms and fecal slurries as required by FEAST and metaSourceTracker, and kmtricks k-mer count tables per decOM documentation, then run each tool on the mesocosm samples with fecal slurry/source profiles as training data. Run SourceApp at its tuned parameters: 93% minimum read-to-genome alignment identity (-r), 70% minimum alignment-to-read overlap (-q), 0% target genome masking, and a limit of detection of 0.01% relative abundance. Score every tool's source attribution/apportionment predictions against the ground-truth cell fractions established during mesocosm construction (Supporting Information Table S1), computing sensitivity, specificity, and correlation with true fractions. Tabulate results side-by-side (e.g., tool_comparison_table.csv); the validated outcome is SourceApp reaching sensitivity 0.902 and specificity 0.898, correlation 0.86 with ground truth, versus 0.79 (FEAST), 0.74 (decOM), and 0.60 (metaSourceTracker), with alternatives showing very poor specificity (0.30 FEAST, 0 decOM and metaSourceTracker).

## Related tools

- **SourceApp** (Benchmarked tool under evaluation; performs source attribution and apportionment by mapping reads to source-associated genome databases at tuned parameters (93% identity, 70% overlap, 0.01% LOD)) — github.com/bglindner/SourceApp
- **FEAST** (Alternative source tracking tool benchmarked on Kraken2 profiles of mesocosm and fecal slurry data)
- **metaSourceTracker** (Alternative source tracking tool benchmarked on Kraken2 profiles of mesocosm and fecal slurry data)
- **decOM** (Alternative k-mer-based source tracking tool benchmarked on kmtricks k-mer count tables)
- **Kraken2** (Generates the taxonomic profiles required by FEAST and metaSourceTracker)
- **kmtricks** (Generates k-mer count tables required as decOM input)
- **CoverM** (Read coverage/relative abundance calculator cited among the article's methods resources) — github.com/wwood/CoverM

## Examples

```
git clone https://github.com/blindner6/SourceApp.git; python sourceapp.py --use-geq mesocosm_R1.fastq.gz mesocosm_R2.fastq.gz
```

## Evaluation signals

- The reproduced tool_comparison_table.csv must show SourceApp sensitivity 0.902 and specificity 0.898, exceeding all alternative tools (FEAST specificity 0.30; decOM and metaSourceTracker specificity 0)
- Apportionment accuracy must rank SourceApp > FEAST > decOM > metaSourceTracker (correlation with ground truth 0.86, 0.79, 0.74, 0.60)
- Predictions must be scored against the ground-truth cell fractions in Supporting Information Table S1 established during mesocosm construction
- SourceApp parameters must match the top-scoring tuned set: 93% identity, 70% alignment-to-read overlap, 0% target genome masking, 0.01% LOD
- Per-source residuals should be inspected: substantial residuals remain after linear correction even at best parameters (r2 = 0.64), and cow sources show underestimation/false negatives

## Limitations

- SourceApp performance varies substantially by source; cow fecal microbiomes are undersampled in the database, leading to underestimation and false negatives for cows across samples
- SourceApp portions estimate proportions of cells based on genome equivalents and do not represent a mass balance; they may differ from mass-based expectations
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations, and persistence of species-level marker genomes in the environment is largely unknown
- SourceApp works best when users supplement the default genome reference database with additional reference genomes; alternatives (FEAST, metaSourceTracker, decOM) require sequencing of potential source samples
- Substantial residuals remain after linear correction of cell fraction estimates even with the best-performing parameters (r2 = 0.64)

## Evidence

- [abstract] Parameter tuning revealed sensitivity and specificity near 0.90 overall, which exceeded all alternative tools.: "Parameter tuning revealed sensitivity and specificity near 0.90 overall, which exceeded all alternative tools."
- [results] Tuned SourceApp parameter set and resulting metrics: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] MAG quality threshold applied when building source databases: "genomes with aggregate quality scores below 50% as calculated by quality = completeness −5 × redundancy were discarded"
- [results] Benchmark novelty statement grounding the comparison design: "no work exists which has benchmarked metagenomic FST methodologies against gold standard data sets such as the spike-in mesocosms"
- [readme] SourceApp input contract from README: "detect and quantify fecal signal in metagenomes collected from the water environment if users will simply provide gzipped copies of the raw reads in FASTQ format."
