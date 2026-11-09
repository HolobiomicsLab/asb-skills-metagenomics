---
name: metatranscriptome-read-subsampling
description: Decision support for planning metatranscriptome sequencing depth, or
  when validating that a sample's annotated-read count suffices for accurate low-abundance
  transcript quantification.
when_to_use_negative:
- The goal is taxonomic/functional differential analysis of already-sequenced samples
  (use the SAMSA aggregation and R DESeq2 analysis scripts instead) rather than estimating
  required sequencing depth
- Samples are expected to yield far fewer than 5-10 million annotated reads and the
  aim is not to quantify the resulting accuracy loss
- Analysis targets organism population size shifts where metatranscriptomics is known
  to be less accurate than shotgun metagenomics
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: SAMSA
  role: 'Pipeline context: pilot sample is annotated via MG-RAST and aggregated into
    summary files that serve as the reference abundances for subsampling'
  repo: https://github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server producing the annotation results from which aggregated organism/functional
    summaries are derived
- name: Python 2.7
  role: Loads aggregated summary files and performs the random per-depth subsampling
    and accuracy calculations
- name: R (RStudio)
  role: Imports per-depth accuracy results and plots accuracy versus sequencing depth
    stratified by abundance class
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:61b803415671b3f2a76f89d8a3c639f415f739721ed7ca24da0d25c9ac3c6104
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - MG-RAST
  - Python 2.7
  - R
  - R (RStudio)
  - SAMSA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metatranscriptome-read-subsampling

## Summary

A rarefaction-style simulation that repeatedly subsamples annotated metatranscriptome reads at increasing depths to determine the minimum sequencing depth required for accurate transcript abundance measurement, stratified by transcript abundance class (high/medium/low).

## When to use

Decision support for planning metatranscriptome sequencing depth, or when validating that a sample's annotated-read count suffices for accurate low-abundance transcript quantification.

## When NOT to use

- The goal is taxonomic/functional differential analysis of already-sequenced samples (use the SAMSA aggregation and R DESeq2 analysis scripts instead) rather than estimating required sequencing depth
- Samples are expected to yield far fewer than 5-10 million annotated reads and the aim is not to quantify the resulting accuracy loss
- Analysis targets organism population size shifts where metatranscriptomics is known to be less accurate than shotgun metagenomics

## Inputs

- Aggregated organism/functional summary files from MG-RAST annotation of a deeply sequenced pilot metatranscriptome sample
- Full-sample reference transcript relative abundances
- Subsampling grid specification (1M-20M annotated reads, 10% increments, 10 replicates per size)

## Outputs

- Per-depth, per-abundance-class relative-abundance accuracy table
- Accuracy-versus-sequencing-depth rarefaction curve plot stratified by transcript abundance class (R/RStudio)
- Recommended minimum sequencing depth (annotated and raw read counts) for >90% low-abundance accuracy

## How to apply

Start from aggregated organism/functional summary files derived from MG-RAST annotation of a deeply sequenced pilot sample, loaded with Python 2.7. For each subsampling size from 1M to 20M annotated reads in 10% increments, draw 10 random subsets of annotated reads per size. For each subset, compute transcript relative abundances and stratify transcripts into high-, medium-, and low-abundance classes based on their abundance in the full sample. Compute relative-abundance accuracy per class per depth against the full-sample reference abundances, then import per-depth results into R (RStudio) and plot accuracy versus sequencing depth stratified by abundance class. Mark the depth at which low-abundance accuracy exceeds 90%; the paper's validated expectation is that this occurs at 5-10 million annotated reads, equivalent to about 40-50 million raw sequence reads.

## Related tools

- **SAMSA** (Pipeline context: pilot sample is annotated via MG-RAST and aggregated into summary files that serve as the reference abundances for subsampling) — https://github.com/transcript/SAMSA
- **MG-RAST** (Annotation server producing the annotation results from which aggregated organism/functional summaries are derived)
- **Python 2.7** (Loads aggregated summary files and performs the random per-depth subsampling and accuracy calculations)
- **R (RStudio)** (Imports per-depth accuracy results and plots accuracy versus sequencing depth stratified by abundance class)

## Evaluation signals

- The accuracy-vs-depth curve must show low-abundance transcript accuracy exceeding 90% at 5-10 million annotated reads (about 40-50 million raw sequences), matching the paper's reported recommendation
- At every subsampling size, 10 independent random subsets are drawn; per-size accuracy should be reported over these replicates, with tighter variance at larger depths
- High- and medium-abundance transcript classes should reach high accuracy at shallower depths than the low-abundance class
- Each sample needs 40-50 million raw sequences expected to yield 5-10 million annotated reads; if the pilot's annotation yield ratio differs substantially, re-derive the raw-read equivalent

## Limitations

- MG-RAST annotation servers require queue waits of days to weeks, which delays obtaining the annotated input needed for the simulation
- Metatranscriptome mRNA counts cannot measure microbial population size shifts as accurately as shotgun metagenomics, so the derived depth applies to expression/activity measurement, not organism abundance profiling
- The 40-50 million raw read recommendation was derived for ribodepleted stool metatranscriptomes sequenced in 100 bp paired-end format; other sample types or read formats may need re-estimation

## Evidence

- [abstract] Each sample needs 40-50 million raw sequences, expected to yield 5-10 million annotated reads for accurate abundance measures: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [abstract] recommendation for ribodepleted, 100 bp paired-end sequencing with minimum 40 million reads: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
- [methods] SAMSA dependencies include Python 2.7 and R/RStudio, the tools used for subsampling and plotting: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R, preferably RStudio"
- [readme] SAMSA pipeline inputs are raw FASTQ files processed beginning to end: "Starting with a raw sequenced file in fastq format, it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)."
