---
name: throughput-metrics-extraction
description: Use when you need to quantify the computational efficiency of a multi-rule
  bioinformatics workflow — e.g. per-rule memory footprints, wall-clock times, and
  Gbp/h throughput across a sample set.
when_to_use_negative:
- The pipeline supports only short-read sequencing data, so benchmarking throughput
  for long-read (PacBio/Nanopore) datasets is out of scope
- You only need functional output validation (e.g. ARG abundance tables) rather than
  computational resource accounting
- The workflow is not Snakemake-based and does not emit per-rule benchmark logs, so
  per-rule metrics cannot be parsed
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: Snakemake
  role: Workflow manager whose rule-level benchmark logs supply wall-clock time, peak
    memory, and CPU-hour measurements
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastq-dl
  role: Downloads the ENA benchmark read datasets (e.g. PRJEB34633) into raw_reads
    for the throughput run
  repo: https://github.com/rpetit3/fastq-dl
- name: ARGprofiler
  role: The Snakemake pipeline being benchmarked; its rules (fastp, KMA, ARGextender,
    Mash) are the units of measurement
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastp
  role: Trimming/QC rule whose per-sample output basepair counts feed the Gbp/h throughput
    calculation
  repo: https://github.com/OpenGene/fastp
- name: Mash
  role: Sketching rule (sketch size 1e4, k-mer size 31) included in the per-rule resource
    profile
  repo: https://github.com/marbl/Mash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:fcda66362e42c6343140c9fe372bf0927fc440060263cfc41e1d4aea3c869d3f
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# throughput-metrics-extraction

## Summary

Benchmark a Snakemake metagenomic pipeline by parsing rule-level benchmark logs into per-rule wall-clock time, peak memory, and CPU-hours, and by computing sample-level processing throughput in Gbp/h to characterize time, cost, and energy expenditure of large-scale analysis.

## When to use

Use when you need to quantify the computational efficiency of a multi-rule bioinformatics workflow — e.g. per-rule memory footprints, wall-clock times, and Gbp/h throughput across a sample set.

## When NOT to use

- The pipeline supports only short-read sequencing data, so benchmarking throughput for long-read (PacBio/Nanopore) datasets is out of scope
- You only need functional output validation (e.g. ARG abundance tables) rather than computational resource accounting
- The workflow is not Snakemake-based and does not emit per-rule benchmark logs, so per-rule metrics cannot be parsed

## Inputs

- Metagenomic FASTQ read datasets retrieved from ENA (e.g. Copenhagen sewage collection under PRJEB34633) via fastq-dl into a raw_reads directory
- Snakemake per-rule benchmark logs (wall-clock time, max RSS memory, CPU-hours) from a full pipeline run
- Read-length/basepair counts per sample for Gbp/h throughput computation

## Outputs

- Per-rule resource table: wall-clock time, sample-average peak memory footprint, and CPU-hour consumption
- Per-sample throughput in Gbp/h with cross-sample average (1.21 Gbp/h) and median (0.36 Gbp/h)
- Flagged list of resource-heavy rules exceeding the <1 GB / <1 CPU-hour envelope (KMA-mOTUs, ARGextender)

## How to apply

Run the workflow end-to-end (here: ARGprofiler Snakemake pipeline with fastp trimming, KMA alignment against mOTUs3 and PanRes, ARGextender recursive KMA + SPAdes extension, and Mash sketching at sketch size 1e4 and k-mer size 31) with Snakemake's benchmark logging enabled. Parse the per-rule benchmark logs to extract wall-clock time, sample-average peak memory footprint, and CPU-hour consumption for each rule. Compute total basepairs processed per hour per sample and derive both the average and median throughput across the benchmark set (reported values: 1.21 Gbp/h average, 0.36 Gbp/h median). Tabulate rules whose sample-average peak memory exceeds 1 GB or that consume more than one CPU hour — in ARGprofiler these were KMA-mOTUs and ARGextender, while most other rules stayed below 1 GB peak memory.

## Related tools

- **Snakemake** (Workflow manager whose rule-level benchmark logs supply wall-clock time, peak memory, and CPU-hour measurements) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads the ENA benchmark read datasets (e.g. PRJEB34633) into raw_reads for the throughput run) — https://github.com/rpetit3/fastq-dl
- **ARGprofiler** (The Snakemake pipeline being benchmarked; its rules (fastp, KMA, ARGextender, Mash) are the units of measurement) — https://github.com/genomicepidemiology/ARGprofiler
- **fastp** (Trimming/QC rule whose per-sample output basepair counts feed the Gbp/h throughput calculation) — https://github.com/OpenGene/fastp
- **Mash** (Sketching rule (sketch size 1e4, k-mer size 31) included in the per-rule resource profile) — https://github.com/marbl/Mash

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Computed average throughput reproduces the reported 1.21 Gbp/h and median reproduces 0.36 Gbp/h within a reasonable tolerance
- Every rule has a populated benchmark log entry with wall-clock time and peak memory before tabulation
- Exactly the KMA-mOTUs and ARGextender rules are flagged as exceeding the <1 GB / <1 CPU-hour envelope, matching the paper's finding
- Summed CPU-hours across rules is consistent with total wall-clock time and the configured thread counts per rule
- Gbp/h per-sample values derive from total basepairs divided by total pipeline wall-clock time, cross-checked against fastp QC reports

## Limitations

- Average and median throughput diverge strongly (1.21 vs 0.36 Gbp/h), indicating skewed sample sizes; a few large samples dominate the mean
- Memory outliers (KMA-mOTUs, ARGextender) mean the <1 GB claim does not generalize to the heaviest rules
- Benchmark results depend on the execution host (e.g. HPC module environment) and thread allocation, so absolute numbers are not portable across systems

## Evidence

- [results] With its final parameter settings, ARGprofiler processed 1.21 Gbp/h on average with a median of 0.36 Gbp/h, and most steps had a sample-average peak memory footprint below 1 GB: [evidence span withheld — anchor status: not_found]
- [intro] workflow efficiency matters: [evidence span withheld — anchor status: not_found]
- [abstract] Snakemake implementation: "The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data"
- [discussion] short-read limitation: "ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies"
- [readme] Snakefile invocation from benchmark README: [evidence span withheld — anchor status: external_doc]
