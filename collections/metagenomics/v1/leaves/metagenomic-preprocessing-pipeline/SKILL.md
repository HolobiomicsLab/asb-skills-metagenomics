---
name: metagenomic-preprocessing-pipeline
description: Use when raw metagenomic samples arrive as SRA archives and must be converted
  to FASTQ/FASTA, quality-checked, and characterized with per-sample sequence statistics
  before taxonomic/functional mapping.
when_to_use_negative:
- Input reads are already in FASTQ/FASTA and only need mapping — skip SRA conversion
  and go straight to the mapping stage of CAMAMED
- The goal is differential abundance analysis or compositional normalization (CSS/metagenomeSeq)
  — that is a later CAMAMED stage operating on abundance tables, not on raw reads
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: SRA Toolkit
  role: fastq-dump (v2.8.2) converts SRA archives to FASTQ/FASTA, keeping paired-end
    reads separated
  repo: https://github.com/mhnb/camamed
- name: FastQC
  role: v0.11.5 quality control of converted FASTQ/FASTA files (per-base quality,
    GC content, adapter contamination)
  repo: https://github.com/mhnb/camamed
- name: SeqKit
  role: v0.10.1 seqkit stats computes per-sample sequence statistics collected into
    seqkit_stats.tsv
  repo: https://github.com/mhnb/camamed
- name: CAMAMED
  role: Pipeline whose pre-processing stage encapsulates the conversion, QC, and statistics
    steps; distributed as Docker images
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Docker images (camamed_pipeline, camamed_pipeline_db) remove the need to install
    preprocessing dependencies separately
  repo: https://github.com/mhnb/camamed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:d8f38b45e16b1b67eeaa1a9b20ddb4dc35710bc17309e4b7793b22262bb643fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - FastQC
  - SRA Toolkit
  - SeqKit
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# metagenomic-preprocessing-pipeline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-processes raw metagenomic sequencing archives (SRA) into FASTQ/FASTA with fastq-dump (SRA Toolkit v2.8.2), then assesses read quality with FastQC v0.11.5 and extracts per-sample sequence statistics with SeqKit v0.10.1, as done in the CAMAMED pipeline before mapping.

## When to use

Use when raw metagenomic samples arrive as SRA archives and must be converted to FASTQ/FASTA, quality-checked, and characterized with per-sample sequence statistics before taxonomic/functional mapping.

## When NOT to use

- Input reads are already in FASTQ/FASTA and only need mapping — skip SRA conversion and go straight to the mapping stage of CAMAMED
- The goal is differential abundance analysis or compositional normalization (CSS/metagenomeSeq) — that is a later CAMAMED stage operating on abundance tables, not on raw reads

## Inputs

- Metagenomic sample archives in SRA format
- Converted FASTQ/FASTA read files (paired-end reads kept separated)

## Outputs

- Converted FASTQ/FASTA read files per sample
- Per-sample FastQC HTML quality-control reports (per-base quality, GC content, adapter contamination)
- seqkit_stats.tsv: tab-separated table of per-sample read counts and sequence length min/mean/max

## How to apply

Retrieve each metagenomic sample in SRA format and convert it to FASTQ/FASTA using fastq-dump from SRA Toolkit version 2.8.2, keeping paired-end reads separated. Run FastQC v0.11.5 on every converted FASTQ/FASTA file to obtain per-sample quality-control reports covering per-base quality scores, GC content, and adapter contamination. Then use SeqKit v0.10.1 (seqkit stats) to compute per-sample statistics such as read counts and sequence length min/mean/max. Collect the SeqKit statistics into a single tab-separated file (seqkit_stats.tsv) and retain the FastQC HTML reports alongside the converted FASTQ/FASTA files as inputs to the downstream CAMAMED mapping steps. All tools are provided without separate installation via the CAMAMED Docker images (camamed_pipeline and camamed_pipeline_db).

## Related tools

- **SRA Toolkit** (fastq-dump (v2.8.2) converts SRA archives to FASTQ/FASTA, keeping paired-end reads separated) — https://github.com/mhnb/camamed
- **FastQC** (v0.11.5 quality control of converted FASTQ/FASTA files (per-base quality, GC content, adapter contamination)) — https://github.com/mhnb/camamed
- **SeqKit** (v0.10.1 seqkit stats computes per-sample sequence statistics collected into seqkit_stats.tsv) — https://github.com/mhnb/camamed
- **CAMAMED** (Pipeline whose pre-processing stage encapsulates the conversion, QC, and statistics steps; distributed as Docker images) — https://github.com/mhnb/camamed
- **Docker** (Docker images (camamed_pipeline, camamed_pipeline_db) remove the need to install preprocessing dependencies separately) — https://github.com/mhnb/camamed

## Examples

```
fastq-dump --split-files SRR_sample.sra && fastqc SRR_sample_1.fastq SRR_sample_2.fastq && seqkit stats *.fastq > seqkit_stats.tsv
```

## Evaluation signals

- A seqkit_stats.tsv table exists and contains one row per sample with read counts and sequence length min/mean/max
- Every sample has a corresponding FastQC HTML report; per-base quality scores, GC content, and adapter contamination sections are populated
- Paired-end samples produced two read files (kept separated) rather than interleaved output
- Converted files are valid FASTQ/FASTA (e.g. parseable by seqkit stats without errors)
- Downstream mapping steps of CAMAMED receive the retained FASTQ/FASTA files without further conversion

## Limitations

- The pipeline pins specific tool versions (SRA Toolkit 2.8.2, FastQC 0.11.5, SeqKit 0.10.1); other versions are not validated
- The article text does not state software versions or dataset accessions in the sections provided; version details come from the task card rather than the paper sections
- Pre-processing only prepares and characterizes reads — it does not correct compositional bias, which CAMAMED addresses later with cumulative sum-scaling at taxa and gene levels

## Evidence

- [other] quality control of the sequence dataset is performed using FastQC v0.11.5, and the statistical characteristics of samples are extracted using SeqKit v0.10.1: [evidence span withheld — anchor status: not_found]
