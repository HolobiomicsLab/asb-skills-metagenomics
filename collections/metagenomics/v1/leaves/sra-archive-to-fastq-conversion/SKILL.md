---
name: sra-archive-to-fastq-conversion
description: Use when input metagenomic samples arrive as NCBI SRA archives and must
  be converted to FASTQ/FASTA and quality-checked before mapping-based taxonomic or
  functional profiling in the CAMAMED pipeline.
when_to_use_negative:
- Input samples are already in FASTQ/FASTA format and only need QC statistics rather
  than SRA conversion
- The goal is downstream CAMAMED analysis (mapping to non-redundant gene catalogs,
  CSS normalization, KEGG/KEGG ortholog annotation) rather than pre-processing raw
  SRA archives
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: SRA Toolkit
  role: fastq-dump (v2.8.2) converts each SRA archive to FASTQ/FASTA with paired-end
    reads separated
- name: FastQC
  role: v0.11.5 generates per-sample QC reports (per-base quality scores, GC content,
    adapter contamination) on the converted files
- name: SeqKit
  role: v0.10.1 stats computes per-sample read counts, sequence length min/mean/max,
    and GC content assembled into seqkit_stats.tsv
- name: CAMAMED
  role: Pipeline whose pre-processing stage this skill implements, feeding converted
    reads into the downstream mapping steps
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Pre-packaged camamed_pipeline / camamed_pipeline_db images avoid installing
    SRA Toolkit, FastQC, or SeqKit dependencies separately
  repo: https://hub.docker.com
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:cf076b4afd70d10b6a1b02770c3d8e7af065d146d1d5157b5a085a363ca51b6e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# SRA archive to FASTQ conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-processes metagenomic sample archives by converting SRA files to FASTQ/FASTA with SRA Toolkit's fastq-dump, then validating each converted file with FastQC v0.11.5 quality reports and SeqKit v0.10.1 sequence statistics before the CAMAMED mapping stage.

## When to use

Use when input metagenomic samples arrive as NCBI SRA archives and must be converted to FASTQ/FASTA and quality-checked before mapping-based taxonomic or functional profiling in the CAMAMED pipeline.

## When NOT to use

- Input samples are already in FASTQ/FASTA format and only need QC statistics rather than SRA conversion
- The goal is downstream CAMAMED analysis (mapping to non-redundant gene catalogs, CSS normalization, KEGG/KEGG ortholog annotation) rather than pre-processing raw SRA archives

## Inputs

- Metagenomic sample files in SRA archive format
- SRA Toolkit 2.8.2 (fastq-dump)
- FastQC v0.11.5
- SeqKit v0.10.1

## Outputs

- Per-sample converted FASTQ/FASTA read files with paired-end reads kept separated
- Per-sample FastQC HTML quality-control reports (per-base quality, GC content, adapter contamination)
- Tab-separated sequence statistics table named seqkit_stats.tsv

## How to apply

Retrieve the metagenomic sample files in SRA format and convert each archive to FASTQ/FASTA using the fastq-dump tool from SRA Toolkit (version 2.8.2 as specified in the pipeline), keeping paired-end reads separated. Run FastQC v0.11.5 on every converted FASTQ/FASTA file to generate per-sample quality-control reports covering per-base quality scores, GC content, and adapter contamination. Use SeqKit stats (v0.10.1) to compute per-sample read counts and sequence length min/mean/max plus GC content. Collect the SeqKit statistics into a single tab-separated table named seqkit_stats.tsv and retain the FastQC HTML reports alongside the converted FASTQ/FASTA files for downstream mapping steps. The rationale is that the CAMAMED mapping stage requires per-sample read files, and QC reports let you catch adapter contamination or poor per-base quality before sequences are mapped to non-redundant gene catalogs.

## Related tools

- **SRA Toolkit** (fastq-dump (v2.8.2) converts each SRA archive to FASTQ/FASTA with paired-end reads separated)
- **FastQC** (v0.11.5 generates per-sample QC reports (per-base quality scores, GC content, adapter contamination) on the converted files)
- **SeqKit** (v0.10.1 stats computes per-sample read counts, sequence length min/mean/max, and GC content assembled into seqkit_stats.tsv)
- **CAMAMED** (Pipeline whose pre-processing stage this skill implements, feeding converted reads into the downstream mapping steps) — https://github.com/mhnb/camamed
- **Docker** (Pre-packaged camamed_pipeline / camamed_pipeline_db images avoid installing SRA Toolkit, FastQC, or SeqKit dependencies separately) — https://hub.docker.com

## Examples

```
fastq-dump --split-files sample.sra && fastqc sample_1.fastq sample_2.fastq && seqkit stats sample_1.fastq sample_2.fastq >> seqkit_stats.tsv
```

## Evaluation signals

- Each input .sra file yields a converted FASTQ/FASTA file, with paired-end reads kept as separate files
- A FastQC HTML report exists for every converted sample and includes per-base quality scores, GC content, and adapter contamination modules
- seqkit_stats.tsv exists, is tab-separated, and contains read counts plus min/mean/max sequence lengths and GC content for each sample
- The number of rows in seqkit_stats.tsv matches the number of converted sample files
- Converted FASTQ/FASTA files, FastQC reports, and seqkit_stats.tsv are all retained together as inputs for the mapping stage

## Limitations

- No software version is stated for CAMAMED itself or its component tools in the article text; only FastQC v0.11.5, SeqKit v0.10.1, and SRA Toolkit 2.8.2 are version-pinned in the task workflow
- This skill covers only the pre-processing stage; it does not perform the mapping to non-redundant gene catalogs or the CSS compositional-bias correction of the later CAMAMED stages
- Installing dependencies manually can be error-prone; the README notes 'There is no need to install any software dependencies if you use the Docker image'

## Evidence

- [full_text] In the pre-processing stage, quality control of the sequence dataset is performed using FastQC v0.11.5, and the statistical characteristics of samples are extracted using SeqKit v0.10.1.: [evidence span withheld — anchor status: not_found]
- [supplementary] Retrieve the metagenomic sample files in SRA format and convert each archive to FASTQ/FASTA using the fastq-dump tool from SRA Toolkit (version 2.8.2 as specified in the pipeline): [evidence span withheld — anchor status: not_found]
