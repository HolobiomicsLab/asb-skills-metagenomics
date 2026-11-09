---
name: computational-resource-profiling
description: Use when you need to quantify throughput (Gbp/h) and per-rule wall-clock
  time, peak memory, or CPU-hours of a Snakemake metagenomic workflow to judge its
  cost/energy feasibility.
when_to_use_negative:
- The pipeline is not Snakemake-based and does not emit per-rule benchmark logs —
  log-parsing throughput profiling will not apply
- 'Input is long-read sequencing data: ARGprofiler is designed to only work with short-read
  data, so its resource profile cannot be generalized'
- The goal is accuracy/quality benchmarking (e.g. taxonomic profiler accuracy on CAMI
  data) rather than computational cost profiling
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow engine whose rule-level benchmark logs supply the per-rule timing,
    memory, and CPU-hour measurements
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastq-dl
  role: Downloads the ENA/SRA benchmark read sets (e.g. PRJEB34633) into raw_reads
    before profiling the pipeline
  repo: https://github.com/rpetit3/fastq-dl
- name: fastp
  role: Trimming/QC rule whose read-pair counts after filtering feed the basepairs-per-hour
    throughput calculation
  repo: https://github.com/OpenGene/fastp
- name: KMA
  role: Alignment rule against mOTUs3 and PanRes; the KMA-mOTUs rule is one of the
    flagged >1 GB memory exceptions
- name: ARGextender
  role: Assembly rule (recursive KMA + SPAdes) that is the other flagged exception
    exceeding the <1 GB / <1 CPU-hour envelope
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: Mash
  role: Sketching rule (sketch size 1e4, k-mer size 31) whose lightweight resource
    profile is expected to fall under the 1 GB envelope
  repo: https://github.com/marbl/Mash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:b5181cb52504efaded8bb9d887a8e0ebaebb704c1a9fc86f6e971f5ed23bb8a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - Mash
  - PanRes
  - Snakemake
  - fastp
  - fastq-dl
  - mOTUs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# computational-resource-profiling

## Summary

Benchmark a Snakemake-based metagenomic pipeline (e.g. ARGprofiler) by parsing rule-level benchmark logs to quantify per-rule wall-clock time, peak memory, and CPU-hour consumption, then report throughput in Gbp/h. This determines whether the workflow is efficient enough for large-scale reuse of public metagenomic data.

## When to use

Use when you need to quantify throughput (Gbp/h) and per-rule wall-clock time, peak memory, or CPU-hours of a Snakemake metagenomic workflow to judge its cost/energy feasibility.

## When NOT to use

- The pipeline is not Snakemake-based and does not emit per-rule benchmark logs — log-parsing throughput profiling will not apply
- Input is long-read sequencing data: ARGprofiler is designed to only work with short-read data, so its resource profile cannot be generalized
- The goal is accuracy/quality benchmarking (e.g. taxonomic profiler accuracy on CAMI data) rather than computational cost profiling

## Inputs

- Metagenomic FASTQ reads downloaded from ENA/SRA (e.g. PRJEB34633) into a raw_reads directory
- Snakemake workflow (ARGprofiler Snakefile) with final parameter settings
- Snakemake per-rule benchmark log files (wall-clock time, peak memory, CPU-hours)

## Outputs

- Per-rule resource table (wall-clock time, peak memory footprint, CPU-hour consumption per rule)
- Per-sample throughput in Gbp/h with average and median across the benchmark set
- Flagged list of resource-heavy rules (sample-average peak memory >1 GB or >1 CPU-hour)

## How to apply

Retrieve the raw sequencing reads (e.g. the Copenhagen sewage collection under ENA project PRJEB34633) with fastq-dl into a raw_reads directory, then run the ARGprofiler Snakemake workflow end-to-end with the paper's final parameter settings: fastp trimming/QC, KMA alignment against mOTUs3 and PanRes, ARGextender (recursive KMA + SPAdes with >90% query identity, >90% global consensus identity, and mean read depth >6 filtering), and Mash sketching (sketch size 1e4, k-mer size 31). Parse the Snakemake rule-level benchmark logs to extract per-rule wall-clock time, peak memory footprint, and CPU-hour consumption. Compute basepairs processed per hour per sample and derive the average and median throughput across the benchmark set. Tabulate the per-rule resource profile and flag rules whose sample-average peak memory exceeds 1 GB or that consume more than one CPU hour. Validation compares computed throughput against reported values (1.21 Gbp/h average, 0.36 Gbp/h median) and confirms only KMA-mOTUs and ARGextender exceed the <1 GB / <1 CPU-hour envelope of the remaining rules.

## Related tools

- **Snakemake** (Workflow engine whose rule-level benchmark logs supply the per-rule timing, memory, and CPU-hour measurements) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads the ENA/SRA benchmark read sets (e.g. PRJEB34633) into raw_reads before profiling the pipeline) — https://github.com/rpetit3/fastq-dl
- **fastp** (Trimming/QC rule whose read-pair counts after filtering feed the basepairs-per-hour throughput calculation) — https://github.com/OpenGene/fastp
- **KMA** (Alignment rule against mOTUs3 and PanRes; the KMA-mOTUs rule is one of the flagged >1 GB memory exceptions)
- **ARGextender** (Assembly rule (recursive KMA + SPAdes) that is the other flagged exception exceeding the <1 GB / <1 CPU-hour envelope) — https://github.com/genomicepidemiology/ARGprofiler
- **Mash** (Sketching rule (sketch size 1e4, k-mer size 31) whose lightweight resource profile is expected to fall under the 1 GB envelope) — https://github.com/marbl/Mash

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Computed average throughput matches the reported 1.21 Gbp/h and median matches 0.36 Gbp/h
- Only KMA-mOTUs and ARGextender rules exceed the <1 GB peak-memory / <1 CPU-hour envelope; all other rules fall below it
- Every parsed benchmark log maps to a valid Snakemake rule and sample, with no missing rule-sample entries
- Total basepairs per sample is consistent with fastp trimming/QC output read counts after filtering
- Per-rule wall-clock times sum consistently with the end-to-end workflow runtime

## Limitations

- Peak-memory exceptions (KMA-mOTUs, ARGextender) mean the <1 GB envelope holds only for most steps, not all — reproducers must budget extra RAM for those rules
- Throughput metrics are sample-set specific (Copenhagen sewage dataset); other datasets will shift average and median Gbp/h
- ARGprofiler only supports short-read data, so resource profiles do not transfer to long-read pipelines

## Evidence

- [abstract] The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data: "The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data"
- [intro] Scalable workflow implemented in Snakemake: "The pipeline is an easily usable and scalable workflow implemented in Snakemake"
- [discussion] ARGextender much shorter runtime than SPAdes: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [readme] Snakemake invocation from repository README: [evidence span withheld — anchor status: external_doc]
- [readme] fastp role as fast preprocessing tool for benchmarked QC step: [evidence span withheld — anchor status: external_doc]
