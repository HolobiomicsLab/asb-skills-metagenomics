---
name: read-type-aware-tool-gating
description: Use when building or modifying an nf-core/mag (Nextflow DSL2) workflow
  that mixes short-read, long-read, and hybrid metagenomic data and each module must
  be routed only to the read types it supports.
when_to_use_negative:
- The workflow handles a single uniform read type with tools that are already valid
  for it, so no gating logic is needed
- Input is not metagenomic sequencing data routed through nf-core/mag modules (e.g.
  already-assembled contigs without associated reads)
- You need tool selection based on data content (e.g. quality thresholds) rather than
  read type — gating by read type alone will not enforce quality cut-offs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: nf-core/mag
  role: Pipeline whose DSL2 modules implement the read-type gating across short, long,
    and hybrid data
  repo: https://github.com/nf-core/mag
- name: Bowtie2
  role: Short-read alignment backend of the gate (vs minimap2 for long reads)
- name: minimap2
  role: Long-read alignment backend of the gate
- name: Porechop
  role: Adapter trimming applied to Oxford Nanopore reads but skipped for PacBio reads
  repo: https://github.com/rrwick/Porechop
- name: Filtlong
  role: Long-read quality filtering routed only to long-read input
  repo: https://github.com/rrwick/Filtlong
- name: GTDB-Tk
  role: Taxonomic classification gated to BUSCO quality-passing MAG bins
- name: CAT/BAT
  role: Fallback taxonomic classifier applied to the remaining bins without marker-gene
    requirements
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:e3fe56e999925657abf428fa391cacee656cfba2c754db9d89560c13e97ae009
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - CAT/BAT
  - Centrifuge
  - Filtlong
  - GTDB-Tk
  - Kraken2
  - MetaBAT2
  - Porechop
  - minimap2
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# Read-type-aware tool gating

## Summary

Gates each nf-core/mag analysis tool to the sequencing read type it supports (short, long, or hybrid), because not every tool is valid for every read type. Ensures e.g. Bowtie2/Porechop/fastp handle short or Nanopore reads while minimap2/Filtlong handle long reads, and that downstream stages (binning, taxonomic classification) only receive eligible inputs.

## When to use

Use when building or modifying an nf-core/mag (Nextflow DSL2) workflow that mixes short-read, long-read, and hybrid metagenomic data and each module must be routed only to the read types it supports.

## When NOT to use

- The workflow handles a single uniform read type with tools that are already valid for it, so no gating logic is needed
- Input is not metagenomic sequencing data routed through nf-core/mag modules (e.g. already-assembled contigs without associated reads)
- You need tool selection based on data content (e.g. quality thresholds) rather than read type — gating by read type alone will not enforce quality cut-offs

## Inputs

- Raw short-read FASTQ files (e.g. Illumina paired-end)
- Raw long-read FASTQ/FASTA files (Oxford Nanopore or PacBio)
- Preprocessed short-read FASTQ
- MAG bin FASTA files with BUSCO quality summaries (completeness/contamination)

## Outputs

- Read-type-routed alignment BAMs (Bowtie2 for short reads, minimap2 for long reads)
- Adapter-trimmed long reads (Porechop, Nanopore only) and quality-filtered long reads (Filtlong)
- Preprocessed short reads plus Krona classification charts (Kraken2/Centrifuge, short reads only)
- mag_taxonomy.tsv with one row per MAG, a taxonomy column and a method column distinguishing GTDB-Tk vs CAT/BAT

## How to apply

Classify each input channel by read type (Illumina short reads vs Nanopore/PacBio long reads vs hybrid), then gate each module to its valid read type: use Bowtie2 for short-read alignment and minimap2 for long-read alignment; run Porechop adapter trimming on Nanopore reads but skip it for PacBio reads; filter long reads with Filtlong and preprocess short reads with fastp. Apply the same gating downstream: Kraken2/Centrifuge read classification runs on preprocessed short reads only, and GTDB-Tk taxonomic classification is applied only to MAG bins that pass BUSCO-derived completeness/contamination thresholds while CAT/BAT covers the remaining bins. The rationale is that nf-core/mag processes short-read, long-read, and hybrid data, and not every tool is valid for every read type, so unfiltered routing would crash modules or produce invalid results.

## Related tools

- **nf-core/mag** (Pipeline whose DSL2 modules implement the read-type gating across short, long, and hybrid data) — https://github.com/nf-core/mag
- **Bowtie2** (Short-read alignment backend of the gate (vs minimap2 for long reads))
- **minimap2** (Long-read alignment backend of the gate)
- **Porechop** (Adapter trimming applied to Oxford Nanopore reads but skipped for PacBio reads) — https://github.com/rrwick/Porechop
- **Filtlong** (Long-read quality filtering routed only to long-read input) — https://github.com/rrwick/Filtlong
- **GTDB-Tk** (Taxonomic classification gated to BUSCO quality-passing MAG bins)
- **CAT/BAT** (Fallback taxonomic classifier applied to the remaining bins without marker-gene requirements)

## Examples

```
nextflow run nf-core/mag --input samplesheet.csv --outdir results -profile docker  # samplesheet rows carry read-type so each tool only receives its valid reads
```

## Evaluation signals

- Short-read samples never enter minimap2 or Porechop processes; long-read samples never enter Bowtie2 or fastp short-read preprocessing
- PacBio reads bypass Porechop adapter trimming while Nanopore reads pass through it
- mag_taxonomy.tsv has GTDB-Tk assignments only for bins meeting the BUSCO completeness/contamination cut-offs; all other bins carry CAT/BAT annotations
- nf-test assertions on bundled test data pass for every read type (short, long, hybrid) with non-empty bin FASTA and per-sample abundance tables
- Version and MultiQC channel conventions hold: version outputs are mixed into ch_versions and relevant files into ch_multiqc for each gated module

## Limitations

- Not every tool applies to every read type, so adding a new module requires explicit read-type gating or the workflow fails on unsupported inputs
- In nf-core/mag 2.1.0 functional annotation and bin refinement steps were not yet included, so gating cannot route those stages
- Porechop adapter trimming is skipped for PacBio reads, so PacBio-specific adapter issues need another remedy

## Evidence

- [methods] Porechop adapter trimming is skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [methods] short- vs long-read alignment uses Bowtie2 vs minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [abstract] It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning"
- [readme] fastp short-read scoping from README: [evidence span withheld — anchor status: not_found]
- [readme] Porechop scoping from README: [evidence span withheld — anchor status: external_doc]
