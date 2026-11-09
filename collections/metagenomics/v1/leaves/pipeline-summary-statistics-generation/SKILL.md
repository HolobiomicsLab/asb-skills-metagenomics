---
name: pipeline-summary-statistics-generation
description: Use when running MetaLAFFA on shotgun metagenomic FASTQs and you need
  per-sample, per-step quality metrics (e.g. post-trim average base quality, average
  read length, number of reads) to audit QC and annotation quality.
when_to_use_negative:
- When input data are already feature/abundance tables rather than raw or intermediate
  FASTQ/BAM pipeline outputs — there are no reads to summarize
- When you need alignment-level or functional-profile metrics (e.g. UniRef90 match
  rates) rather than per-step read/length/quality summaries — use the corresponding
  downstream analysis instead
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaLAFFA
  role: Pipeline whose Snakemake steps automatically generate and merge per-sample
    summary statistics after each pre-processing and annotation operation
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Trimmomatic
  role: Quality trimming tool whose MAXINFO-based quality_filter output FASTQs (60bp
    min length) are the source of post-trim average base quality and read length statistics
- name: Snakemake
  role: Workflow engine that orchestrates the per-step summary statistic generation
    and table merging
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:3f70a6e0df084916e0bcefe10504366bc79a394329f0216c756f8f4acefff9d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Snakemake
  - Trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pipeline-summary-statistics-generation

## Summary

MetaLAFFA automatically generates per-sample summary statistics for each step of its Snakemake-based metagenomic functional annotation pipeline (e.g. read counts, average read length, and average base quality after Trimmomatic MAXINFO quality trimming), merging them into step-level summary tables so users can assess pre-processing and annotation quality.

## When to use

Use when running MetaLAFFA on shotgun metagenomic FASTQs and you need per-sample, per-step quality metrics (e.g. post-trim average base quality, average read length, number of reads) to audit QC and annotation quality.

## When NOT to use

- When input data are already feature/abundance tables rather than raw or intermediate FASTQ/BAM pipeline outputs — there are no reads to summarize
- When you need alignment-level or functional-profile metrics (e.g. UniRef90 match rates) rather than per-step read/length/quality summaries — use the corresponding downstream analysis instead

## Inputs

- Per-sample output files from each MetaLAFFA step (e.g. gzipped quality-filtered FASTQs following <sample_id>.<type_id>.fastq.gz naming)
- Host-filtered paired and singleton FASTQs from the initial data directory

## Outputs

- Per-sample summary statistics (number of reads, average read length, average base quality) for each pipeline step
- Merged per-step summary table across samples

## How to apply

After each pipeline step (host filtering, duplicate filtering, quality trimming/filtering, mapping, gene/ortholog counting), MetaLAFFA's summary step computes per-sample statistics from the step's output files — for example, after the Trimmomatic MAXINFO quality_filter step (60bp minimum length threshold), it reports number of reads, average read length, and average base quality of the trimmed FASTQs. These statistics are written per sample and merged into a summary table for the step, giving users visibility into how many reads/bases each pre-processing operation removed. Interpret the metrics by comparing pre- and post-step values: large read loss in quality trimming or host filtering flags aggressive parameters, while near-zero discard (as for the post-HMP quality-control HMP stool samples) indicates the QC phase was conservative. Use the merged tables as validation artifacts confirming each step produced its expected output before downstream annotation proceeds.

## Related tools

- **MetaLAFFA** (Pipeline whose Snakemake steps automatically generate and merge per-sample summary statistics after each pre-processing and annotation operation) — https://github.com/borenstein-lab/MetaLAFFA
- **Trimmomatic** (Quality trimming tool whose MAXINFO-based quality_filter output FASTQs (60bp min length) are the source of post-trim average base quality and read length statistics)
- **Snakemake** (Workflow engine that orchestrates the per-step summary statistic generation and table merging)

## Evaluation signals

- For each sample, the step summary table contains non-null values for number of reads, average read length, and average base quality
- Post-trim average base quality in the summary table exceeds the quality threshold enforced during Trimmomatic MAXINFO trimming
- Post-trim average read length is at or above the 60bp minimum length threshold
- Per-sample read counts in the summary table are monotonically non-increasing across sequential filtering steps (host filtering → duplicate filtering → quality trimming)
- The trimmed gzipped FASTQ (<sample_id>.<type_id>.fastq.gz) exists in the output directory and no reads below 60bp are present

## Limitations

- Minimal reads were discarded during the quality control phase for data already post-HMP quality control, so summary statistics may show near-zero filtering effect for pre-cleaned data and provide little QC signal
- Summary statistics are limited to per-step read-level metrics (count, length, base quality); they do not quantify annotation coverage or gene-level quality downstream

## Evidence

- [abstract] Per-step summary statistics are a first-class MetaLAFFA output: "MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality"
- [intro] Quality trimming source of post-trim metrics: "quality trimming and filtering, which removes low-quality bases from the ends of reads and then discards reads that are too short"
- [results] Conservative QC on pre-cleaned HMP samples: "Since these data files are post-HMP quality control, minimal reads were discarded from each sample during MetaLAFFA’s quality control phase."
- [readme] Summary step is a named step in the tutorial workflow: "10. Summarizing results of steps in MetaLAFFA"
