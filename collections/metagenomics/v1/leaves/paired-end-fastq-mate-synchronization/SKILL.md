---
name: paired-end-fastq-mate-synchronization
description: Use when raw paired-end shotgun viral metagenomic FASTQ files (R1/R2)
  need adapter removal, low-quality base trimming, and host/contaminant read removal
  before viral identification, and mates must stay in register.
when_to_use_negative:
- Input reads are already cleaned — ViroProfiler skips this step in that case
- Data are single-end reads, since the paired-end adapter detection (--detect_adapter_for_pe)
  and R1/R2 synchronization logic do not apply
- Samples with excessive environmental contaminations, which per the article increase
  pipeline running time and reduce precision — isolate viral fractions before sequencing
  instead
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline that orchestrates the read-cleaning (quality-control)
    step at the start of viral metagenomic analysis
  repo: https://github.com/deng-lab/viroprofiler
- name: FASTP
  role: Performs adapter trimming (-f 15 -t 1 -F 15 -T 1 --detect_adapter_for_pe),
    low-quality base removal, polyG/polyX trimming, and drops reads shorter than 30
    bp
- name: BBMap
  role: DECONTAM filter quality-trims both ends at Q10 (qtrim=rl trimq=10) and removes
    contaminant reads with at least 2 k-mer hits (minhits=2)
- name: FastQC
  role: Optional QC sanity check on the cleaned reads (run with --quiet)
- name: Nextflow
  role: Workflow engine used to launch ViroProfiler's quality-control processes (e.g.
    with the docker,test profile)
- name: Docker
  role: Containerization engine that packages the cleaning tools for reproducible
    execution
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:e46ec9498917df2e602cc5350b09e17fc9af1225f628a9dca8986b39c2c388be
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Paired-end FASTQ mate synchronization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A read-cleaning skill from the ViroProfiler pipeline that trims and filters shotgun metagenomic FASTQ reads with FASTP and the BBMap DECONTAM filter while keeping R1/R2 mates synchronized, producing cleaned paired-end FASTQ files per sample.

## When to use

Use when raw paired-end shotgun viral metagenomic FASTQ files (R1/R2) need adapter removal, low-quality base trimming, and host/contaminant read removal before viral identification, and mates must stay in register.

## When NOT to use

- Input reads are already cleaned — ViroProfiler skips this step in that case
- Data are single-end reads, since the paired-end adapter detection (--detect_adapter_for_pe) and R1/R2 synchronization logic do not apply
- Samples with excessive environmental contaminations, which per the article increase pipeline running time and reduce precision — isolate viral fractions before sequencing instead

## Inputs

- Paired-end shotgun viral metagenomic FASTQ files (R1/R2) per sample
- Optional contaminant/host reference k-mer index for the BBMap DECONTAM filter

## Outputs

- Cleaned paired-end FASTQ files (R1/R2) per sample with mates synchronized
- Optional FASTQC reports on the cleaned reads

## How to apply

Launch the ViroProfiler quality-control processes via Nextflow (e.g. 'nextflow run denglab/viroprofiler -profile docker,test'). Run FASTP with the ViroProfiler defaults '-f 15 -t 1 -F 15 -T 1 --detect_adapter_for_pe -p -n 1 -l 30 -5 -W 4 -M 20 -r -c -g -x': this trims 15 bases from the front and tail of each read, auto-detects adapters for paired-end data, removes low-quality bases, polyG/polyX tails, and drops reads shorter than 30 bp. Then run the DECONTAM (BBMap) filter with 'maxindel=1 bwr=0.16 bw=12 quickmatch fast minhits=2 qtrim=rl trimq=10 pigz=True untrim' to quality-trim both ends at Q10 and remove contaminant reads with at least 2 k-mer hits. Optionally run FASTQC with '--quiet' on the cleaned reads as a QC sanity check. The step is skipped when the input reads are already cleaned. Write cleaned R1/R2 FASTQ files per sample, ensuring mates remain synchronized through every filter so both mates of a pair survive or are discarded together.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline that orchestrates the read-cleaning (quality-control) step at the start of viral metagenomic analysis) — https://github.com/deng-lab/viroprofiler
- **FASTP** (Performs adapter trimming (-f 15 -t 1 -F 15 -T 1 --detect_adapter_for_pe), low-quality base removal, polyG/polyX trimming, and drops reads shorter than 30 bp)
- **BBMap** (DECONTAM filter quality-trims both ends at Q10 (qtrim=rl trimq=10) and removes contaminant reads with at least 2 k-mer hits (minhits=2))
- **FastQC** (Optional QC sanity check on the cleaned reads (run with --quiet))
- **Nextflow** (Workflow engine used to launch ViroProfiler's quality-control processes (e.g. with the docker,test profile))
- **Docker** (Containerization engine that packages the cleaning tools for reproducible execution)

## Examples

```
nextflow run denglab/viroprofiler -profile docker,test
```

## Evaluation signals

- The cleaned paired-end FASTQ files (R1 and R2) exist for each sample
- Cleaned outputs are in valid FASTQ format
- R1 and R2 contain equal numbers of reads, confirming mate synchronization after trimming and contaminant filtering
- Reads shorter than 30 bp are absent following FASTP's '-l 30' filter
- Optional FASTQC report on cleaned reads shows no adapter or per-base quality failures

## Limitations

- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision; the paper recommends isolating viral fractions before sequencing
- The cleaning step is skipped when input reads are already cleaned, so pre-cleaned inputs bypass FASTP/DECONTAM parameters entirely
- Mate synchronization must be preserved manually or by the pipeline; running the tools with mismatched R1/R2 would break paired-end downstream analysis

## Evidence

- [other] The pipeline's cleaning step processes input reads by removing adapters, low quality reads and contaminant reads, and this step is skipped when the input reads are already cleaned.: [evidence span withheld — anchor status: not_found]
- [other] Run FASTP with the ViroProfiler default arguments '-f 15 -t 1 -F 15 -T 1 --detect_adapter_for_pe -p -n 1 -l 30 -5 -W 4 -M 20 -r -c -g -x': [evidence span withheld — anchor status: not_found]
- [other] Run the DECONTAM (BBMap) filter with arguments 'maxindel=1 bwr=0.16 bw=12 quickmatch fast minhits=2 qtrim=rl trimq=10 pigz=True untrim': [evidence span withheld — anchor status: not_found]
