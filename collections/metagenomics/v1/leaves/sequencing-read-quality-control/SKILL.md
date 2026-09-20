---
name: sequencing-read-quality-control
description: Use when raw metagenomic samples arrive as SRA archives and must be converted
  to FASTQ/FASTA, quality-checked, and summarized with sequence statistics before
  mapping in the CAMAMED pipeline.
when_to_use_negative:
- Input reads are already in FASTQ/FASTA form and have been converted from SRA — only
  the FastQC and SeqKit steps apply
- The analysis goal is downstream compositional analysis (CSS normalization, KEGG
  annotation, biomarker identification) rather than pre-mapping read QC
- Working with already-assembled contigs or gene-frequency tables instead of raw sequencing
  reads
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: SRA Toolkit
  role: fastq-dump (v2.8.2) converts SRA archives to FASTQ/FASTA, keeping paired-end
    reads separated
- name: FastQC
  role: v0.11.5 generates per-sample quality-control reports for each converted FASTQ/FASTA
    file
- name: SeqKit
  role: v0.10.1 seqkit stats computes per-sample read counts and sequence length statistics
    collected into seqkit_stats.tsv
- name: CAMAMED
  role: parent pipeline whose pre-processing stage this skill implements before mapping
    reads to non-redundant gene catalogs
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: camamed_pipeline and camamed_pipeline_db images bundle the pipeline dependencies,
    removing the need to install tools separately
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:8c477efee7b93cfdcd8f5d6e1a48c7a988184dbf688bfcbe88d837e21773c2a3
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sequencing-read-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs the CAMAMED pre-processing stage: converting SRA archives to FASTQ/FASTA with fastq-dump (SRA Toolkit v2.8.2), running FastQC v0.11.5 quality control on each converted read file, and extracting per-sample sequence statistics with SeqKit v0.10.1.

## When to use

Use when raw metagenomic samples arrive as SRA archives and must be converted to FASTQ/FASTA, quality-checked, and summarized with sequence statistics before mapping in the CAMAMED pipeline.

## When NOT to use

- Input reads are already in FASTQ/FASTA form and have been converted from SRA — only the FastQC and SeqKit steps apply
- The analysis goal is downstream compositional analysis (CSS normalization, KEGG annotation, biomarker identification) rather than pre-mapping read QC
- Working with already-assembled contigs or gene-frequency tables instead of raw sequencing reads

## Inputs

- Metagenomic sample files in SRA archive format
- Converted FASTQ/FASTA read files (paired-end reads kept separated)

## Outputs

- Per-sample FastQC v0.11.5 HTML quality-control reports (per-base quality, GC content, adapter contamination)
- seqkit_stats.tsv — a single tab-separated table of per-sample read counts and sequence length min/mean/max from SeqKit v0.10.1
- Converted FASTQ/FASTA read files retained for downstream CAMAMED mapping steps

## How to apply

Retrieve the metagenomic sample files in SRA format and convert each archive to FASTQ/FASTA using the fastq-dump tool from SRA Toolkit version 2.8.2, keeping paired-end reads separated. Run FastQC v0.11.5 on every converted FASTQ/FASTA file to generate per-sample quality-control reports covering per-base quality scores, GC content, and adapter contamination. Use SeqKit v0.10.1 (seqkit stats) to compute per-sample read counts and sequence length min/mean/max statistics, collecting them into a single tab-separated table named seqkit_stats.tsv. Retain the FastQC HTML reports alongside the converted FASTQ/FASTA files for the downstream mapping steps of CAMAMED; running the pipeline via the camamed_pipeline Docker images avoids installing these dependencies individually.

## Related tools

- **SRA Toolkit** (fastq-dump (v2.8.2) converts SRA archives to FASTQ/FASTA, keeping paired-end reads separated)
- **FastQC** (v0.11.5 generates per-sample quality-control reports for each converted FASTQ/FASTA file)
- **SeqKit** (v0.10.1 seqkit stats computes per-sample read counts and sequence length statistics collected into seqkit_stats.tsv)
- **CAMAMED** (parent pipeline whose pre-processing stage this skill implements before mapping reads to non-redundant gene catalogs) — https://github.com/mhnb/camamed
- **Docker** (camamed_pipeline and camamed_pipeline_db images bundle the pipeline dependencies, removing the need to install tools separately) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
fastq-dump --split-files SRRXXXXX.sra && fastqc *.fastq && seqkit stats -T *.fastq > seqkit_stats.tsv
```

## Evaluation signals

- seqkit_stats.tsv exists and is a tab-separated table with one row per sample including read counts and sequence length min/mean/max
- A FastQC HTML report exists for every converted FASTQ/FASTA file
- Paired-end samples yield separated read files after fastq-dump conversion
- Converted FASTQ/FASTA files and FastQC reports are retained together for the downstream mapping steps of CAMAMED
- Tool versions match the pipeline specification: SRA Toolkit 2.8.2, FastQC v0.11.5, SeqKit v0.10.1

## Limitations

- The CAMAMED paper text and README do not state software versions for the pipeline's component tools, so version pinning relies on the pipeline specification (SRA Toolkit 2.8.2, FastQC v0.11.5, SeqKit v0.10.1)
- Docker images come in two variants — camamed_pipeline (without MetaPhlAn2 databases) and camamed_pipeline_db (with databases) — so the correct image must be chosen for taxonomic profiling downstream
- No changelog is available for the CAMAMED repository

## Evidence

- [methods] quality control of the sequence dataset is performed using FastQC v0.11.5, and the statistical characteristics of samples are extracted using SeqKit v0.10.1: [evidence span withheld — anchor status: not_found]
- [methods] convert each archive to FASTQ/FASTA using the fastq-dump tool from SRA Toolkit (version 2.8.2 as specified in the pipeline), keeping paired-end reads separated: [evidence span withheld — anchor status: not_found]
