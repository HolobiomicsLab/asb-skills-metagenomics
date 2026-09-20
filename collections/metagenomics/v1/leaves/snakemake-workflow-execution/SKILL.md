---
name: snakemake-workflow-execution
description: Use when metagenomic FASTQ runs (e.g. ENA PRJEB34633 sewage reads) must
  be processed for ARG abundance, flanking reconstruction and Mash sketching, and
  when rule-level wall-clock, peak-memory, and Gbp/h performance metrics are needed.
when_to_use_negative:
- Input is long-read (Nanopore/PacBio) data — ARGprofiler is designed to only work
  with short-read sequencing data
- The goal is a standalone taxonomy or ARG call on a single sample rather than scalable,
  reusable large-scale pipeline execution with resource profiling
- The target gene may be absent and an expensive full assembly must run regardless
  — the ARGextender design deliberately skips assembly when no target genes are identified
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ARGprofiler
  role: The Snakemake workflow being executed; orchestrates fastp, KMA, ARGextender,
    and Mash steps
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: Snakemake
  role: Workflow engine that runs the rules and emits per-rule benchmark logs used
    for resource profiling
- name: fastq-dl
  role: Downloads raw FASTQ runs from ENA/SRA accessions into the raw_reads directory
    before execution
  repo: https://github.com/rpetit3/fastq-dl
- name: fastp
  role: Trimming and quality control rule applied to raw reads
  repo: https://github.com/OpenGene/fastp
- name: KMA
  role: Aligns reads against mOTUs3 and PanRes to produce abundance tables and seed
    ARGextender
- name: ARGextender
  role: Recursive KMA + SPAdes rule that reconstructs ARG flanking regions with >90%
    query identity, >90% global consensus identity, mean read depth >6
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: SPAdes
  role: De novo assembler invoked by ARGextender on recruited reads
- name: Mash
  role: Creates sketches (size 1e4, k 31) for fast k-mer-based comparison of sequencing
    runs
  repo: https://github.com/marbl/Mash
- name: PanRes
  role: Combined ARG reference database (14,078 unique ARGs) targeted by KMA/ARGextender
    rules
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:2b12163cf6edb2281fd255855096ec9d47d405466f00fbb03216a1e549dccd03
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
  - SPAdes
  - Snakemake
  - fastp
  - fastq-dl
  - mOTUs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# snakemake-workflow-execution

## Summary

Execute the ARGprofiler Snakemake workflow end-to-end on metagenomic short-read data with the paper's final parameter settings, then harvest per-rule benchmark logs to quantify throughput (Gbp/h) and per-rule CPU/memory footprints.

## When to use

Use when metagenomic FASTQ runs (e.g. ENA PRJEB34633 sewage reads) must be processed for ARG abundance, flanking reconstruction and Mash sketching, and when rule-level wall-clock, peak-memory, and Gbp/h performance metrics are needed.

## When NOT to use

- Input is long-read (Nanopore/PacBio) data — ARGprofiler is designed to only work with short-read sequencing data
- The goal is a standalone taxonomy or ARG call on a single sample rather than scalable, reusable large-scale pipeline execution with resource profiling
- The target gene may be absent and an expensive full assembly must run regardless — the ARGextender design deliberately skips assembly when no target genes are identified

## Inputs

- Raw metagenomic short-read FASTQ files downloaded via fastq-dl into a raw_reads directory (e.g. ENA runs from PRJEB34633)
- PanRes ARG database (14,078 unique ARGs) for KMA alignment and ARGextender
- mOTUs3 microbial reference database for KMA-based profiling
- Snakemake profile/configfile specifying the paper's final parameter settings

## Outputs

- Gene and microbe abundance tables (KMA .mapstat-derived, e.g. via kma2cami-style parsing)
- Targeted de novo assemblies of ARG flanking regions from ARGextender (KMA + SPAdes)
- Mash sketches (sketch size 1e4, k-mer size 31) per sequencing run
- Per-rule benchmark logs (wall-clock time, peak memory, CPU-hours) and derived throughput values (Gbp/h per sample, average and median)

## How to apply

Retrieve raw reads with fastq-dl (rpetit3/fastq-dl) into a raw_reads directory from an ENA accession such as the Copenhagen sewage project PRJEB34633. Run the ARGprofiler Snakemake pipeline (genomicepidemiology/ARGprofiler) end-to-end so all rules fire: fastp trimming/QC, KMA alignment against mOTUs3 and PanRes, ARGextender (recursive KMA + SPAdes recruiting reads around target ARGs, filtering at >90% query identity, >90% global consensus identity, and mean read depth >6), and Mash sketching (sketch size 1e4, k-mer size 31). Parse the Snakemake rule-level benchmark logs to extract wall-clock time, peak memory, and CPU-hour consumption per rule, and compute basepairs processed per hour per sample to derive average and median throughput. Because ARGextender only runs after KMA confirms target genes are present, the design avoids the expensive assembly step when no target ARG is identified. Validate by comparing computed throughput to the reported 1.21 Gbp/h average / 0.36 Gbp/h median and confirming only KMA-mOTUs and ARGextender exceed the <1 GB / <1 CPU-hour envelope.

## Related tools

- **ARGprofiler** (The Snakemake workflow being executed; orchestrates fastp, KMA, ARGextender, and Mash steps) — https://github.com/genomicepidemiology/ARGprofiler
- **Snakemake** (Workflow engine that runs the rules and emits per-rule benchmark logs used for resource profiling)
- **fastq-dl** (Downloads raw FASTQ runs from ENA/SRA accessions into the raw_reads directory before execution) — https://github.com/rpetit3/fastq-dl
- **fastp** (Trimming and quality control rule applied to raw reads) — https://github.com/OpenGene/fastp
- **KMA** (Aligns reads against mOTUs3 and PanRes to produce abundance tables and seed ARGextender)
- **ARGextender** (Recursive KMA + SPAdes rule that reconstructs ARG flanking regions with >90% query identity, >90% global consensus identity, mean read depth >6) — https://github.com/genomicepidemiology/ARGprofiler
- **SPAdes** (De novo assembler invoked by ARGextender on recruited reads)
- **Mash** (Creates sketches (size 1e4, k 31) for fast k-mer-based comparison of sequencing runs) — https://github.com/marbl/Mash
- **PanRes** (Combined ARG reference database (14,078 unique ARGs) targeted by KMA/ARGextender rules)

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Computed average throughput across the benchmark set matches 1.21 Gbp/h and median matches 0.36 Gbp/h
- Per-rule benchmark logs show sample-average peak memory below 1 GB for all rules except KMA-mOTUs and ARGextender
- Only KMA-mOTUs and ARGextender exceed the <1 GB / <1 CPU-hour envelope satisfied by the remaining rules
- All rules (fastp, KMA against mOTUs3 and PanRes, ARGextender, Mash sketching) executed and produced their expected outputs for each sample
- ARGextender assembly is skipped for samples where KMA identifies no target ARGs

## Limitations

- ARGprofiler is designed to only work with short-read sequencing data, so long-read inputs cannot be processed by this workflow
- KMA-mOTUs and ARGextender require substantially more memory/CPU than the other rules (exceeding the <1 GB / <1 CPU-hour envelope)
- ARGextender's read recruitment is bounded by its thresholds (>90% query identity, >90% global consensus identity, mean read depth >6), which may miss distant or low-coverage flanking regions

## Evidence

- [abstract] The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data: "The ARGprofiler pipeline is a Snakemake workflow that supports the reuse of metagenomic sequencing data"
- [abstract] ARGextender combines KMA and SPAdes to recruit reads for targeted de novo assembly of ARG flanking regions: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] This approach also avoids running the more expensive algorithm if none of the target genes is identified: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [readme] Snakemake invocation from the benchmarking README: [evidence span withheld — anchor status: external_doc]
- [readme] fastq-dl description as FASTQ downloader from ENA/SRA: [evidence span withheld — anchor status: external_doc]
