---
name: sequencing-depth-saturation-analysis
description: Use when planning or justifying sequencing depth for shotgun metatranscriptomes
  (e.g., human gut stool samples), or when asked to demonstrate via subsampling how
  many annotated reads are required to reach a target accuracy for low-abundance transcripts.
when_to_use_negative:
- Input data has not been annotated yet (raw FASTQ without MG-RAST or DIAMOND annotation
  results) — subsampling requires annotated reads, not raw sequences
- The analysis goal is differential expression between control vs experimental groups
  (use the SAMSA R/DESeq2 scripts instead of depth saturation analysis)
- Only very few reads are available overall — a shallow pilot sample cannot serve
  as the full-sample reference for stratifying high/medium/low abundance classes
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: SAMSA
  role: Pipeline whose MG-RAST-annotated, aggregated summary outputs provide the read
    inventory being subsampled for the depth-accuracy curve
  repo: https://github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server that assigns organism/function annotations to reads, yielding
    the annotated reads counted for the 5-10 million read-depth recommendation
- name: Python 2.7
  role: Performs the random subsampling at each depth (1M-20M annotated reads, 10%
    increments, 10 subsets each) and computes per-class relative-abundance accuracies
- name: R (RStudio)
  role: Imports per-depth accuracy results and plots accuracy-versus-depth curves
    stratified by abundance class, marking the >90% threshold
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:7172aa41042a338ed0910c1f100e4a288adf0897b81e7ab97b4ea17fc20c0ca3
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sequencing-depth-saturation-analysis

## Summary

A rarefaction-style saturation analysis that subsamples annotated metatranscriptome reads at increasing depths to find the minimum sequencing depth needed for accurate abundance measurement, especially for low-abundance transcripts. In the SAMSA paper this established that 5-10 million annotated reads (40-50 million raw reads) yield >90% accuracy.

## When to use

Use when planning or justifying sequencing depth for shotgun metatranscriptomes (e.g., human gut stool samples), or when asked to demonstrate via subsampling how many annotated reads are required to reach a target accuracy for low-abundance transcripts.

## When NOT to use

- Input data has not been annotated yet (raw FASTQ without MG-RAST or DIAMOND annotation results) — subsampling requires annotated reads, not raw sequences
- The analysis goal is differential expression between control vs experimental groups (use the SAMSA R/DESeq2 scripts instead of depth saturation analysis)
- Only very few reads are available overall — a shallow pilot sample cannot serve as the full-sample reference for stratifying high/medium/low abundance classes

## Inputs

- Aggregated organism/functional summary files derived from MG-RAST annotation of a deeply sequenced pilot metatranscriptome sample
- Full-sample reference transcript relative abundances (for class stratification and accuracy computation)
- Raw FASTQ reads as the upstream input (100 bp paired-end, ribodepleted stool metatranscriptome)

## Outputs

- Per-depth, per-abundance-class accuracy table (subsample size vs relative-abundance accuracy, 10 replicates per size)
- Plot of accuracy versus sequencing depth stratified by high/medium/low abundance class, with the >90% low-abundance accuracy depth marked (5-10M annotated reads / 40-50M raw reads)

## How to apply

Load the aggregated organism/functional summary files produced from MG-RAST annotation of a deeply sequenced pilot sample using Python 2.7. For each subsampling size (1M to 20M annotated reads in 10% increments), draw 10 random subsets of annotated reads, compute transcript relative abundances per subset, and stratify transcripts into high-, medium-, and low-abundance classes based on their abundance in the full sample. Compute relative-abundance accuracy per class and depth against the full-sample reference abundances, import the per-depth accuracy results into R (RStudio), and plot accuracy versus sequencing depth stratified by abundance class. Mark the depth at which low-abundance accuracy exceeds 90%: in this dataset that point falls at 5-10 million annotated reads, equivalent to ~40-50 million raw sequence reads, matching the paper's recommendation of a minimum of 40 million raw reads per sample.

## Related tools

- **SAMSA** (Pipeline whose MG-RAST-annotated, aggregated summary outputs provide the read inventory being subsampled for the depth-accuracy curve) — https://github.com/transcript/SAMSA
- **MG-RAST** (Annotation server that assigns organism/function annotations to reads, yielding the annotated reads counted for the 5-10 million read-depth recommendation)
- **Python 2.7** (Performs the random subsampling at each depth (1M-20M annotated reads, 10% increments, 10 subsets each) and computes per-class relative-abundance accuracies)
- **R (RStudio)** (Imports per-depth accuracy results and plots accuracy-versus-depth curves stratified by abundance class, marking the >90% threshold)

## Examples

```
import random; subset = random.sample(annotated_reads, k=1000000)  # 10 random subsets per size, 1M-20M annotated reads in 10% increments, accuracies then plotted in R
```

## Evaluation signals

- The accuracy-vs-depth curve must show >90% accuracy for low-abundance transcripts at 5-10 million annotated reads (40-50 million raw sequences), matching the paper's recommendation
- Each subsampling size must include 10 independent random subsets, with accuracy summarized across replicates rather than a single draw
- Subsample sizes must span 1M to 20M annotated reads in 10% increments so the curve brackets the 5-10M threshold
- Transcript abundance classes (high/medium/low) must be defined relative to the full-sample reference abundances, and low-abundance class accuracy should be the limiting curve (i.e., lowest accuracy at shallow depths)
- The recommended depth must be expressed in both annotated reads (5-10M) and raw sequences (~40-50M), consistent with the observed annotation yield

## Limitations

- The 5-10M annotated read threshold is calibrated on a deeply sequenced human gut (stool) pilot sample; other sample types with different transcript complexity may saturate at different depths
- MG-RAST annotation yield depends on database and queue times (days to weeks), and MG-RAST is not tailored to eukaryotic organisms, which may affect annotation rates and thus the annotated-read counts the recommendation is based on
- Sequencing-depth guidance assumes rRNA-depleted samples with remaining ribosomal reads discarded; non-ribodepleted data contain insufficient mRNA reads for comparable accuracy measurement

## Evidence

- [abstract] Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures.: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [abstract] We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample.: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
- [methods] SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R, preferably RStudio.: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R, preferably RStudio."
- [readme] After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2.: "After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2."
