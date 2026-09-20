---
name: relative-abundance-accuracy-assessment
description: Use when planning metatranscriptome sequencing or judging whether a sample's
  depth suffices for accurate abundance measurement, especially of low-abundance transcripts
  in gut microbiome mRNA data.
when_to_use_negative:
- The goal is to measure shifts in microbial population sizes with metagenomics-level
  accuracy — the paper notes metatranscriptomes cannot measure population size shifts
  as accurately as shotgun metagenomics
- The sample is eukaryote-dominated, since MG-RAST is not tailored to work with eukaryotic
  organisms and shows increased mismatch rates
- Input data is already a processed abundance table at target depth — this skill calibrates
  depth, it does not re-quantify abundances
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: MG-RAST
  role: Annotation server that produced the annotation results from which annotated-read
    counts and abundances are derived
  repo: github.com/transcript/SAMSA
- name: SAMSA
  role: Pipeline scripts (e.g., analysis_counter.py) that aggregate raw annotation
    results into the organism/functional summary files used for subsampling
  repo: github.com/transcript/SAMSA
- name: Python 2.7
  role: Performs random subsampling of annotated reads and computes per-class relative-abundance
    accuracies
  repo: github.com/transcript/SAMSA
- name: R (RStudio)
  role: Imports per-depth accuracy results and plots accuracy versus sequencing depth
    stratified by abundance class
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:8e4f05c870dcc42855655b7e6939eeb819471c94dc6ada3e79eaebfb18a49705
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MG-RAST
  - Python 2.7
  - R
  - R (RStudio)
  - SAMSA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# relative-abundance-accuracy-assessment

## Summary

A sequencing-depth calibration skill for metatranscriptomes that subsamples annotated reads at increasing depths, stratifies transcripts by abundance class, and measures relative-abundance accuracy to identify the minimum depth for reliable low-abundance transcript quantification.

## When to use

Use when planning metatranscriptome sequencing or judging whether a sample's depth suffices for accurate abundance measurement, especially of low-abundance transcripts in gut microbiome mRNA data.

## When NOT to use

- The goal is to measure shifts in microbial population sizes with metagenomics-level accuracy — the paper notes metatranscriptomes cannot measure population size shifts as accurately as shotgun metagenomics
- The sample is eukaryote-dominated, since MG-RAST is not tailored to work with eukaryotic organisms and shows increased mismatch rates
- Input data is already a processed abundance table at target depth — this skill calibrates depth, it does not re-quantify abundances

## Inputs

- Aggregated organism and functional summary files (e.g., analysis_counter.py output from MG-RAST annotation results) from a deeply sequenced pilot metatranscriptome sample
- Subsample sizes from 1M to 20M annotated reads in 10% increments
- Full-sample reference relative abundances for all transcripts

## Outputs

- Per-depth relative-abundance accuracy values for high-, medium-, and low-abundance transcript classes
- A plot of accuracy versus sequencing depth stratified by abundance class, with the >90% accuracy threshold depth marked
- A minimum sequencing-depth recommendation (annotated reads and equivalent raw sequence reads)

## How to apply

Load the aggregated organism/functional summary files (derived from MG-RAST annotation results of a deeply sequenced pilot sample) into Python 2.7. For each subsampling size from 1M to 20M annotated reads in 10% increments, draw 10 random subsets of annotated reads, compute transcript relative abundances in each subset, and stratify transcripts into high-, medium-, and low-abundance classes based on their abundance in the full sample. Compare subsample abundances to full-sample reference abundances to compute relative-abundance accuracy per class at each depth. Import the per-depth accuracy results into R (preferably RStudio) and plot accuracy versus sequencing depth stratified by abundance class, marking the depth where low-abundance accuracy exceeds 90%. Per the paper, >90% low-abundance accuracy is reached at 5-10 million annotated reads, equivalent to about 40-50 million raw sequence reads.

## Related tools

- **MG-RAST** (Annotation server that produced the annotation results from which annotated-read counts and abundances are derived) — github.com/transcript/SAMSA
- **SAMSA** (Pipeline scripts (e.g., analysis_counter.py) that aggregate raw annotation results into the organism/functional summary files used for subsampling) — github.com/transcript/SAMSA
- **Python 2.7** (Performs random subsampling of annotated reads and computes per-class relative-abundance accuracies) — github.com/transcript/SAMSA
- **R (RStudio)** (Imports per-depth accuracy results and plots accuracy versus sequencing depth stratified by abundance class) — github.com/transcript/SAMSA

## Examples

```
python analysis_counter.py annotation_results.txt -summary  # aggregate MG-RAST annotations into abundance summaries for subsampling
```

## Evaluation signals

- The accuracy-vs-depth curve must show low-abundance transcript accuracy exceeding 90% at 5-10 million annotated reads
- The recommended depth in raw reads must fall in the 40-50 million raw sequences range, matching the paper's minimum of 40 million reads per sample
- Each subsampling size must be based on 10 random subsets so accuracy estimates reflect sampling variance
- Transcript abundance strata (high/medium/low) must be defined relative to the full-sample reference abundances, not subsample abundances

## Limitations

- MG-RAST annotation servers require queue waits of days to weeks, which slows obtaining the annotated pilot sample needed for subsampling
- The recommendation was derived from human fecal (gut) stool metatranscriptomes ribodepleted and sequenced in 100 bp paired-end format; other sample types or read formats may need different depths
- Sequences must be either longer than 100 bp or joined paired-end reads to yield annotated reads at the expected rate

## Evidence

- [abstract] Each sample needs 40-50 million raw sequences, expected to yield 5-10 million annotated reads for accurate abundance measures: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [abstract] Recommendation of ribodepleted, 100 bp paired-end, minimum 40 million reads per sample: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
- [methods] SAMSA dependencies include Python 2.7 and R (preferably RStudio): "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R, preferably RStudio"
- [readme] README: analysis_counter.py creates a sorted summary of abundance counts from MG-RAST annotation results files: [evidence span withheld — anchor status: external_doc]
