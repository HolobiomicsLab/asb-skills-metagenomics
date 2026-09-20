---
name: read-sequence-statistics-extraction
description: Use when raw metagenomic reads arrive as SRA archives and need conversion
  to FASTQ/FASTA plus per-sample QC reports and sequence statistics before mapping
  to gene catalogs.
when_to_use_negative:
- Input is already a feature/abundance table or gene frequency table rather than raw
  SRA/FASTQ reads — downstream CAMAMED steps (CSS normalization, metagenomeSeq) are
  the appropriate stage
- Data are not metagenomic sequencing reads (e.g. RNA-seq or amplicon FASTQ already
  QC'd by another pipeline)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: SRA Toolkit
  role: fastq-dump (v2.8.2) converts SRA archives to FASTQ/FASTA, keeping paired-end
    reads separated
- name: FastQC
  role: v0.11.5 performs per-sample quality control (per-base quality scores, GC content,
    adapter contamination)
- name: SeqKit
  role: v0.10.1 extracts per-sample sequence statistics (read counts, length min/mean/max,
    GC content) consolidated into seqkit_stats.tsv
- name: Docker
  role: camamed_pipeline and camamed_pipeline_db images ship the pre-processing environment
    so no separate tool installation is needed
- name: CAMAMED
  role: pipeline whose pre-processing stage this skill implements, feeding quality-checked
    reads to mapping-based taxonomic/functional profiling
  repo: https://github.com/mhnb/camamed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:e8ea00bdb5d221103a915d0132ba89676e8682cea7e57fe3ca2578757b1207b1
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# read-sequence-statistics-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-processing stage of the CAMAMED metagenomic pipeline: converting SRA archives to FASTQ/FASTA, running FastQC quality control (v0.11.5), and extracting per-sample sequence statistics (read counts, sequence lengths, GC content) with SeqKit (v0.10.1) before mapping-based analysis.

## When to use

Use when raw metagenomic reads arrive as SRA archives and need conversion to FASTQ/FASTA plus per-sample QC reports and sequence statistics before mapping to gene catalogs.

## When NOT to use

- Input is already a feature/abundance table or gene frequency table rather than raw SRA/FASTQ reads — downstream CAMAMED steps (CSS normalization, metagenomeSeq) are the appropriate stage
- Data are not metagenomic sequencing reads (e.g. RNA-seq or amplicon FASTQ already QC'd by another pipeline)

## Inputs

- Metagenomic sample files in SRA format
- Converted FASTQ/FASTA read files (paired-end reads kept separated)

## Outputs

- FastQC HTML quality-control reports per sample
- seqkit_stats.tsv: tab-separated per-sample table of read counts, sequence length min/mean/max, and GC content
- Converted FASTQ/FASTA read files ready for mapping

## How to apply

Retrieve the metagenomic sample files in SRA format and convert each archive to FASTQ/FASTA with fastq-dump from SRA Toolkit (v2.8.2 in the CAMAMED pipeline), keeping paired-end reads separated. Run FastQC v0.11.5 on every converted file to obtain per-sample quality-control reports covering per-base quality scores, GC content, and adapter contamination. Use SeqKit stats v0.10.1 to compute per-sample read counts and sequence length min/mean/max plus GC content, and collect these into a single tab-separated table named seqkit_stats.tsv. Retain the FastQC HTML reports alongside the converted FASTQ/FASTA files so downstream CAMAMED mapping steps receive quality-checked reads. This normalization-adjacent preprocessing matters because the pipeline's compositional analysis requires reliable abundance estimates; the article notes normalization processes 'identify and eliminate systematic variability and compositional bias'. CAMAMED and its dependencies can be run without separate installation via the Docker images camamed_pipeline and camamed_pipeline_db.

## Related tools

- **SRA Toolkit** (fastq-dump (v2.8.2) converts SRA archives to FASTQ/FASTA, keeping paired-end reads separated)
- **FastQC** (v0.11.5 performs per-sample quality control (per-base quality scores, GC content, adapter contamination))
- **SeqKit** (v0.10.1 extracts per-sample sequence statistics (read counts, length min/mean/max, GC content) consolidated into seqkit_stats.tsv)
- **Docker** (camamed_pipeline and camamed_pipeline_db images ship the pre-processing environment so no separate tool installation is needed)
- **CAMAMED** (pipeline whose pre-processing stage this skill implements, feeding quality-checked reads to mapping-based taxonomic/functional profiling) — https://github.com/mhnb/camamed

## Examples

```
fastq-dump --split-files sample.sra && seqkit stats *.fastq > seqkit_stats.tsv
```

## Evaluation signals

- seqkit_stats.tsv exists and contains one row per sample with read counts, sequence length min/mean/max, and GC content columns
- FastQC HTML reports exist for every converted FASTQ/FASTA sample, covering per-base quality scores, GC content, and adapter contamination
- Paired-end samples produced two read files (e.g. _1/_2) per SRA archive, confirming fastq-dump split the pairs
- Read counts and sequence lengths in seqkit_stats.tsv fall in plausible ranges and are consistent with FastQC's per-sample basic statistics
- Converted FASTQ/FASTA files are retained alongside the QC outputs for downstream CAMAMED mapping steps

## Limitations

- The article/README do not state software versions for CAMAMED itself or minimum quality thresholds (e.g. FastQC per-base quality cutoffs) — QC acceptance criteria must be decided by the practitioner
- Pre-processing quality control is upstream of, and does not replace, compositional-bias correction (CSS normalization with metagenomeSeq), which is applied later at taxa and gene levels
- Using the Docker images is the documented route to avoid dependency installation; running components outside Docker requires installing SRA Toolkit, FastQC and SeqKit separately

## Evidence

- [supplementary] Pre-processing QC and statistics tooling: [evidence span withheld — anchor status: not_found]
- [supplementary] Pre-processing workflow description: [evidence span withheld — anchor status: not_found]
