---
name: metagenomic-pipeline-benchmarking
description: Use when you need to quantify wall-clock time, peak memory footprint,
  and CPU-hour usage of a Snakemake-based metagenomic pipeline (e.g. ARGprofiler with
  fastp, KMA, ARGextender, SPAdes, Mash rules) under its final parameter settings.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler is designed to only work with short-read
  sequencing data
- You need a taxonomy-accuracy comparison rather than resource/throughput benchmarking
  — the separate argprofiler-tax-benchmark repo benchmarks microbial profilers on
  CAMI data instead
- The goal is biological result validation (e.g. flanking-region assembly quality)
  rather than computational cost measurement
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ARGprofiler
  role: Snakemake pipeline being benchmarked; runs fastp, KMA, ARGextender and Mash
    rules
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastq-dl
  role: Downloads ENA read datasets (e.g. PRJEB34633) used as benchmark input
  repo: https://github.com/rpetit3/fastq-dl
- name: fastp
  role: Trimming/QC rule whose per-rule benchmark metrics are collected
  repo: https://github.com/OpenGene/fastp
- name: Mash
  role: Sketching rule (sketch size 1e4, k-mer size 31) included in the resource profile
  repo: https://github.com/marbl/Mash
- name: KMA
  role: Alignment rule against mOTUs3 and PanRes; KMA-mOTUs is flagged as exceeding
    the <1 GB memory envelope
- name: ARGextender
  role: Recursive KMA + SPAdes targeted assembly rule; flagged as a resource-heavy
    step in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:252c60f20d324ba606e471cff930d55f4135713b7155a9d3e19a9b92bc2f0648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - MICCA
  - MICCA-FAST
  - Mash
  - PanRes
  - QIIME
  - RDP
  - UPARSE
  - VSEARCH
  - fastp
  - fastq-dl
  - mOTUs
  - mothur
  - swarm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metagenomic-pipeline-benchmarking

## Summary

Benchmark a large-scale metagenomic workflow (here ARGprofiler, a Snakemake pipeline for ARG analysis) by parsing Snakemake rule-level logs into throughput (Gbp/h), peak memory, and CPU-hour consumption so workflow cost/efficiency can be quantified per rule.

## When to use

Use when you need to quantify wall-clock time, peak memory footprint, and CPU-hour usage of a Snakemake-based metagenomic pipeline (e.g. ARGprofiler with fastp, KMA, ARGextender, SPAdes, Mash rules) under its final parameter settings.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler is designed to only work with short-read sequencing data
- You need a taxonomy-accuracy comparison rather than resource/throughput benchmarking — the separate argprofiler-tax-benchmark repo benchmarks microbial profilers on CAMI data instead
- The goal is biological result validation (e.g. flanking-region assembly quality) rather than computational cost measurement

## Inputs

- Metagenomic FASTQ reads retrieved from ENA via fastq-dl (e.g. PRJEB34633 Copenhagen sewage runs)
- ARGprofiler Snakefile with final parameter settings
- PanRes database (14,078 unique ARGs) and mOTUs3 reference database
- Snakemake per-rule benchmark logs (wall-clock time, peak memory, CPU hours)

## Outputs

- Per-rule resource table (wall-clock time, peak memory footprint, CPU-hour consumption)
- Per-sample and aggregate throughput in Gbp/h (average and median)
- Flagged list of resource-heavy rules (e.g. KMA-mOTUs and ARGextender exceeding 1 GB / 1 CPU-hour)

## How to apply

Retrieve metagenomic read datasets from ENA with fastq-dl (e.g. the Copenhagen sewage collection under PRJEB34633) into a raw_reads directory, then run the ARGprofiler Snakemake workflow end-to-end with the paper's final settings: fastp trimming/QC, KMA alignment against mOTUs3 and PanRes, ARGextender (recursive KMA + SPAdes with >90% query identity, >90% global consensus identity, mean read depth >6 filtering), and Mash sketching (sketch size 1e4, k-mer size 31). Parse the Snakemake rule-level benchmark logs to extract per-rule wall-clock time, peak memory, and CPU-hours. Compute total basepairs processed per hour per sample and derive average and median throughput across the benchmark set. Tabulate per-rule resource profiles and flag rules whose sample-average peak memory exceeds 1 GB or that consume more than one CPU hour; validate against the reported envelope (1.21 Gbp/h average, 0.36 Gbp/h median, with only KMA-mOTUs and ARGextender exceeding 1 GB / 1 CPU-hour).

## Related tools

- **ARGprofiler** (Snakemake pipeline being benchmarked; runs fastp, KMA, ARGextender and Mash rules) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads ENA read datasets (e.g. PRJEB34633) used as benchmark input) — https://github.com/rpetit3/fastq-dl
- **fastp** (Trimming/QC rule whose per-rule benchmark metrics are collected) — https://github.com/OpenGene/fastp
- **Mash** (Sketching rule (sketch size 1e4, k-mer size 31) included in the resource profile) — https://github.com/marbl/Mash
- **KMA** (Alignment rule against mOTUs3 and PanRes; KMA-mOTUs is flagged as exceeding the <1 GB memory envelope)
- **ARGextender** (Recursive KMA + SPAdes targeted assembly rule; flagged as a resource-heavy step in the benchmark)

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Computed average throughput matches the reported 1.21 Gbp/h and median matches 0.36 Gbp/h
- All rules except KMA-mOTUs and ARGextender show sample-average peak memory below 1 GB and below 1 CPU hour
- Per-rule benchmark logs contain non-empty wall-clock, memory, and CPU-hour fields for every executed rule
- Per-sample Gbp/h is reproducible from the parsed benchmark logs and the total basepairs processed per sample

## Limitations

- ARGprofiler is designed to only work with short-read sequencing data, so benchmark results do not transfer to long-read workflows
- Memory and CPU-hour exceptions (KMA-mOTUs, ARGextender) make throughput numbers sensitive to sample composition and hardware
- Throughput metrics were derived on the paper's benchmark set (e.g. Copenhagen sewage runs); other datasets may yield different averages

## Evidence

- [abstract] abstract: "The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data"
- [readme] readme: "A tool for large-scale analysis of antimicrobial resistance genes (ARGs) and their flanking regions in metagenomic datasets."
- [discussion] discussion: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [other] other: [evidence span withheld — anchor status: external_doc]
