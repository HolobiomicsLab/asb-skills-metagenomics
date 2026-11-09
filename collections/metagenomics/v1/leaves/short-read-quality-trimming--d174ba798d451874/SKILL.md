---
name: short-read-quality-trimming
description: Use when raw paired-end shotgun virome FASTQ reads need adapter removal,
  quality trimming, and contaminant filtering as the first stage of ViroProfiler before
  assembly or viral identification.
when_to_use_negative:
- Input reads are already cleaned — the pipeline skips this QC step in that case
- Data are not paired-end shotgun viral metagenome reads (e.g., amplicon data or long
  reads)
- Samples contain excessive environmental contamination that preprocessing cannot
  resolve — the paper recommends isolating viral fractions before sequencing instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline that orchestrates the read-cleaning module
    and manages execution environments
  repo: https://github.com/deng-lab/viroprofiler
- name: FASTP
  role: Performs adapter auto-detection for paired-end data, end/base quality trimming,
    poly-G tail removal, and length filtering (min 30 bp)
- name: BBMap
  role: Provides the DECONTAM filter that removes contaminant reads with at least
    2 k-mer hits and quality-trims both ends at Q10
- name: FastQC
  role: Optional QC sanity check run with '--quiet' on the cleaned reads
- name: Nextflow
  role: Workflow engine used to launch the pipeline's quality-control processes (e.g.,
    with '-profile docker,test')
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:04015c359cd1894cdc12867360eaaa60809aa3c52ea3939291e8e8d7371fe7ff
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
  tools:
  - BBMap
  - Docker
  - FASTP
  - FastQC
  - Nextflow
  - ViroProfiler
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# short-read-quality-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Read-cleaning step at the start of the ViroProfiler viral metagenomics pipeline: FASTP trims adapters and low-quality bases from paired-end FASTQ reads, and an optional BBMap decontamination filter removes host/contaminant reads before downstream viral discovery.

## When to use

Use when raw paired-end shotgun virome FASTQ reads need adapter removal, quality trimming, and contaminant filtering as the first stage of ViroProfiler before assembly or viral identification.

## When NOT to use

- Input reads are already cleaned — the pipeline skips this QC step in that case
- Data are not paired-end shotgun viral metagenome reads (e.g., amplicon data or long reads)
- Samples contain excessive environmental contamination that preprocessing cannot resolve — the paper recommends isolating viral fractions before sequencing instead

## Inputs

- Raw paired-end FASTQ files (R1/R2) per sample from shotgun viral metagenomic sequencing
- Optional host/contaminant reference index for the BBMap decontamination filter

## Outputs

- Cleaned paired-end FASTQ files (R1/R2) per sample with mates kept synchronized
- Optional FASTQC QC reports on the cleaned reads

## How to apply

Launch the pipeline's QC processes via Nextflow with the test profile ('nextflow run denglab/viroprofiler -profile docker,test'). Run FASTP with ViroProfiler defaults '-f 15 -t 1 -F 15 -T 1 --detect_adapter_for_pe -p -n 1 -l 30 -5 -W 4 -M 20 -r -c -g -x': trim the first/last 15 bases, auto-detect adapters for paired-end data, remove low-quality bases, and drop reads shorter than 30 bp. Then run the DECONTAM (BBMap) filter with 'maxindel=1 bwr=0.16 bw=12 quickmatch fast minhits=2 qtrim=rl trimq=10 pigz=True untrim' to quality-trim both ends at Q10 and discard reads with at least 2 contaminant k-mer hits; optionally run FASTQC with '--quiet' as a sanity check. Skip this entire step when the input reads are already cleaned. The rationale is that adapters, low-quality tails, and contaminant reads otherwise inflate downstream runtime and lower precision.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline that orchestrates the read-cleaning module and manages execution environments) — https://github.com/deng-lab/viroprofiler
- **FASTP** (Performs adapter auto-detection for paired-end data, end/base quality trimming, poly-G tail removal, and length filtering (min 30 bp))
- **BBMap** (Provides the DECONTAM filter that removes contaminant reads with at least 2 k-mer hits and quality-trims both ends at Q10)
- **FastQC** (Optional QC sanity check run with '--quiet' on the cleaned reads)
- **Nextflow** (Workflow engine used to launch the pipeline's quality-control processes (e.g., with '-profile docker,test'))

## Examples

```
nextflow run denglab/viroprofiler -profile docker,test
```

## Evaluation signals

- A cleaned paired-end FASTQ pair (R1/R2) exists for every sample
- Each cleaned file is valid FASTQ format and parses without errors
- R1 and R2 contain equal numbers of reads (mates synchronized after trimming/filtering)
- No reads shorter than 30 bp remain after FASTP length filtering
- Reads matching the contaminant reference with >= 2 k-mer hits are absent from cleaned output

## Limitations

- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision
- It is recommended to isolate the viral fractions before sequencing for an accurate estimation of viruses in the samples
- The decontamination step depends on a contaminant reference (e.g., host genome); reads from unindexed contaminants will pass through

## Evidence

- [methods] The pipeline's cleaning step processes input reads by removing adapters, low quality reads and contaminant reads: [evidence span withheld — anchor status: not_found]
