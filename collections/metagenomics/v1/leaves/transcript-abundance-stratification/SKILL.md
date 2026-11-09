---
name: transcript-abundance-stratification
description: 'Decision support for sequencing depth planning: when designing a metatranscriptome
  study and needing to know how many annotated reads are required for accurate low-abundance
  transcript quantification.'
when_to_use_negative:
- Input is already a rarefied or depth-normalized feature table — subsampling-based
  depth calibration is redundant
- The analysis goal is differential expression between control and experimental groups
  (use SAMSA's R/DESeq2 significance scripts instead)
- Samples cannot be sequenced deeply enough to yield a full-sample reference of at
  least ~20M annotated reads, since the depth curve depends on a deeply sequenced
  pilot
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0203
tools:
- name: SAMSA
  role: Pipeline whose MG-RAST annotation outputs and aggregation scripts provide
    the annotated reads and summary files subsampled in this analysis
  repo: github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server that produces the per-read annotations and summary files
    used as the subsampling source
  repo: github.com/transcript/SAMSA
- name: Python 2.7
  role: Used to load summary files, draw random read subsets at each depth, and compute
    stratified abundance accuracies
  repo: github.com/transcript/SAMSA
- name: R
  role: Imports per-depth accuracy results and plots accuracy versus sequencing depth,
    stratified by abundance class
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:24d4a55c435a8cb768d7cd41313f1cecc5b81e80813eea593bcd40753d1b6e47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MG-RAST
  - Python 2.7
  - R
  - SAMSA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# transcript-abundance-stratification

## Summary

Determines the minimum sequencing depth required for accurate metatranscriptome abundance measurements by subsampling annotated reads and stratifying transcripts into high-, medium-, and low-abundance classes to plot accuracy-versus-depth curves.

## When to use

Decision support for sequencing depth planning: when designing a metatranscriptome study and needing to know how many annotated reads are required for accurate low-abundance transcript quantification.

## When NOT to use

- Input is already a rarefied or depth-normalized feature table — subsampling-based depth calibration is redundant
- The analysis goal is differential expression between control and experimental groups (use SAMSA's R/DESeq2 significance scripts instead)
- Samples cannot be sequenced deeply enough to yield a full-sample reference of at least ~20M annotated reads, since the depth curve depends on a deeply sequenced pilot

## Inputs

- Aggregated organism/functional summary files from MG-RAST annotation of a deeply sequenced pilot metatranscriptome sample (e.g. tab-separated abundance summaries)
- Annotated read list from MG-RAST supporting random subsampling at 1M-20M annotated reads

## Outputs

- Per-depth relative-abundance accuracy values stratified by transcript abundance class (high/medium/low)
- Accuracy-versus-sequencing-depth curve (R plot) with the >90% low-abundance accuracy threshold marked
- Recommended minimum sequencing depth in annotated reads and raw sequence equivalents

## How to apply

Load the aggregated organism/functional summary files derived from MG-RAST annotation of a deeply sequenced pilot sample using Python 2.7. For each subsampling size from 1M to 20M annotated reads in 10% increments, draw 10 random subsets of annotated reads per size; for each subset compute transcript relative abundances and stratify transcripts into high-, medium-, and low-abundance classes based on their abundance in the full sample. Compare subsample abundances to the full-sample reference abundances to calculate relative-abundance accuracy per class at each depth, then import the per-depth results into R (RStudio) and plot accuracy versus depth stratified by abundance class. The rationale is that the depth at which low-abundance transcript accuracy exceeds 90% defines the minimum viable sequencing depth; in this study that threshold was reached at 5-10 million annotated reads.

## Related tools

- **SAMSA** (Pipeline whose MG-RAST annotation outputs and aggregation scripts provide the annotated reads and summary files subsampled in this analysis) — github.com/transcript/SAMSA
- **MG-RAST** (Annotation server that produces the per-read annotations and summary files used as the subsampling source) — github.com/transcript/SAMSA
- **Python 2.7** (Used to load summary files, draw random read subsets at each depth, and compute stratified abundance accuracies) — github.com/transcript/SAMSA
- **R** (Imports per-depth accuracy results and plots accuracy versus sequencing depth, stratified by abundance class) — github.com/transcript/SAMSA

## Examples

```
Rscript depth_accuracy_plot.R --input per_depth_accuracy.tsv --strata high,medium,low --threshold 0.90
```

## Evaluation signals

- The accuracy-versus-depth curve must show >90% relative-abundance accuracy for low-abundance transcripts at 5-10 million annotated reads (about 40-50 million raw sequences)
- Each subsampling size must include 10 independent random subsets; the plotted accuracy should reflect variability across these replicates
- Accuracy for high-abundance transcripts should exceed that of low-abundance transcripts at every depth
- Transcript stratification must be defined relative to the full-sample reference abundances, not per-subsample estimates

## Limitations

- The 5-10 million annotated read recommendation was calibrated on human gut (stool) metatranscriptomes with a single pilot sample and may not transfer to other sample types
- MG-RAST annotation yields only a fraction of raw reads as annotated reads (roughly 5-10M annotated from 40-50M raw), so queue waits of days to weeks on MG-RAST servers slow this calibration
- Results assume reads are long (>100 bp) or joined paired-end reads; shorter sequences reduce annotation yield and shift the depth curve

## Evidence

- [abstract] Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures.: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [abstract] Sequencing depth recommendation: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
- [methods] Pipeline dependencies: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R, preferably RStudio."
- [methods] Pipeline purpose: "The purpose of the SAMSA pipeline is to handle beginning-to-end metatranscriptome analysis."
- [discussion] MG-RAST queue limitation: "MG-RAST’s annotation servers require waiting in a queue, and processing may be slowed by days to weeks."
