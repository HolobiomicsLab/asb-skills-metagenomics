---
name: human-and-rrna-read-depletion
description: Use when raw metagenomic FASTQ read sets (e.g., the 264 vaginal metagenomes
  used to build VIRGO) must be cleaned of human-derived reads and rRNA sequences prior
  to assembly, CDS prediction, or mapping-based profiling.
when_to_use_negative:
- Input is already a gene-count/abundance table or assembled contigs — depletion applies
  to raw read FASTQ data
- Data are metatranscriptomic or isolate genome sequences where host/rRNA depletion
  is not the intended preprocessing step
- Paired-end data fed directly to VIRGO mapping scripts without merging — VIRGO does
  not yet support paired-end mapping
tools:
- name: VIRGO
  role: Pipeline whose preprocessing stage removes human-derived and rRNA reads from
    the 264 vaginal metagenomes before assembly and CDS prediction
  repo: github.com/ravel-lab/VIRGO
- name: seqtk
  role: Required FASTQ sequence-manipulation dependency in the VIRGO preprocessing
    chain (seqtk 1.0+)
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Read-alignment engine in the VIRGO dependency chain used for read-mapping
    steps (bowtie 1.1.2+)
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:55b4507134bc28664d4cac9e266cb93044a67a2f51f1b66038b3e051ebd25d4a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VIRGO
  - bowtie
  - seqtk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# human-and-rrna-read-depletion

## Summary

A preprocessing skill from the VIRGO gene-catalog construction workflow: strip human-derived reads and rRNA sequences from vaginal metagenomic FASTQ read sets and apply per-sample QC before de novo assembly and CDS prediction.

## When to use

Use when raw metagenomic FASTQ read sets (e.g., the 264 vaginal metagenomes used to build VIRGO) must be cleaned of human-derived reads and rRNA sequences prior to assembly, CDS prediction, or mapping-based profiling.

## When NOT to use

- Input is already a gene-count/abundance table or assembled contigs — depletion applies to raw read FASTQ data
- Data are metatranscriptomic or isolate genome sequences where host/rRNA depletion is not the intended preprocessing step
- Paired-end data fed directly to VIRGO mapping scripts without merging — VIRGO does not yet support paired-end mapping

## Inputs

- Raw paired-end or single-end metagenomic FASTQ read files (vaginal metagenomes)
- Human reference sequences for host read removal
- rRNA sequence references for ribosomal read removal

## Outputs

- Per-sample quality-filtered FASTQ files depleted of human-derived and rRNA reads
- Per-sample QC reports/metrics on the remaining reads
- Downstream inputs for de novo assembly and CDS prediction (with filtered reads as input)

## How to apply

For each of the vaginal metagenomic read sets, remove reads of human origin and rRNA sequences so that only microbial, protein-coding-derived signal remains; run per-sample QC on the remaining fastq reads to confirm quality and yield before proceeding. Feed the resulting quality-filtered FASTQ files into the downstream VIRGO workflow — de novo per-sample assembly, CDS prediction, and pooling with CDSs from the 308 urogenital isolate genomes, which is de-redundified via all-vs-all Wu-blastp and sequence-similarity clustering into the 948,158-gene non-redundant catalog. Note that VIRGO's mapping scripts expect single-end input, so paired-end reads must be merged into one FASTQ file before processing. Success of this depletion step is ultimately judged by whether the downstream pipeline reproduces the published catalog totals (948,158 non-redundant CDSs, ~17.2% of the original ~5.5 million predictions).

## Related tools

- **VIRGO** (Pipeline whose preprocessing stage removes human-derived and rRNA reads from the 264 vaginal metagenomes before assembly and CDS prediction) — github.com/ravel-lab/VIRGO
- **seqtk** (Required FASTQ sequence-manipulation dependency in the VIRGO preprocessing chain (seqtk 1.0+)) — github.com/ravel-lab/VIRGO
- **bowtie** (Read-alignment engine in the VIRGO dependency chain used for read-mapping steps (bowtie 1.1.2+)) — github.com/ravel-lab/VIRGO

## Evaluation signals

- Per-sample QC metrics on the remaining fastq reads are generated for every sample after depletion
- The quality-filtered reads assemble and support CDS prediction without excessive loss of data
- Downstream de-redundification reproduces the published VIRGO catalog: 948,158 non-redundant CDSs >99 bp, ~17.2% of the original ~5.5 million predicted CDSs, with partition counts 634,288 / 54,500 / 371,099
- Human and rRNA read categories are absent from the retained read pool feeding assembly

## Limitations

- VIRGO mapping scripts do not yet support paired-end input; reads must be merged into a single FASTQ file, which constrains how depleted reads can be fed downstream
- The skill is tied to the reference-based VIRGO approach, whose limitations (bacteria-focused database with limited viral/fungal inclusion) constrain downstream interpretation

## Evidence

- [methods] Preprocess the 264 vaginal metagenomic read sets by removing human-derived reads and rRNA sequences, with per-sample QC on the remaining fastq reads.: [evidence span withheld — anchor status: not_found]
- [readme] Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file.: "Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file."
- [methods] De novo assemble the quality-filtered metagenomic reads per sample and predict coding sequences; simultaneously predict CDSs from the 308 urogenital isolate genomes.: [evidence span withheld — anchor status: not_found]
