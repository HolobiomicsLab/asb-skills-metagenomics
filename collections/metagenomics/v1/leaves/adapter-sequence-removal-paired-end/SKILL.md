---
name: adapter-sequence-removal-paired-end
description: Use when raw paired-end FASTQ reads enter the ViroProfiler pipeline and
  need adapter removal, quality trimming, and contaminant stripping before assembly
  or viral identification; skip only when input reads are already cleaned.
when_to_use_negative:
- Input reads are already cleaned — ViroProfiler's cleaning step is skipped when input
  reads are already cleaned.
- Data are not shotgun viral metagenomic reads (e.g., amplicon or assembled contigs)
  — ViroProfiler is designed for analyzing shotgun viral metagenomic data.
- Samples contain excessive environmental contaminations that were not fractionated
  before sequencing, which could increase running time and result in lower precision;
  isolate the viral fractions before sequencing instead.
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ViroProfiler
  role: Nextflow pipeline whose cleaning step performs adapter, low-quality, and contaminant
    read removal
  repo: https://github.com/deng-lab/viroprofiler
- name: FASTP
  role: Trims first/last 15 bases, auto-detects paired-end adapters, removes low-quality
    bases, drops reads < 30 bp
- name: BBMap (DECONTAM)
  role: Quality-trims both ends at Q10 and removes contaminant reads with >= 2 k-mer
    hits
- name: FastQC
  role: Optional QC sanity check on cleaned reads run with --quiet
- name: Nextflow
  role: Workflow engine launching the quality-control processes with the docker,test
    profile
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:d35d41cc756193ea0bb201426b631cba5dcb7041bf20f46ce0f25dd72337d9e6
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
  - BBMap (DECONTAM)
  - Docker
  - FASTP
  - FastQC
  - Nextflow
  - ViroProfiler
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# adapter-sequence-removal-paired-end

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control of raw paired-end shotgun viral metagenomic FASTQ reads in ViroProfiler, using FASTP trimming (adapter auto-detection, low-quality base removal, short-read dropping) followed by optional DECONTAM (BBMap) host/contaminant filtering, producing cleaned R1/R2 FASTQ pairs.

## When to use

Use when raw paired-end FASTQ reads enter the ViroProfiler pipeline and need adapter removal, quality trimming, and contaminant stripping before assembly or viral identification; skip only when input reads are already cleaned.

## When NOT to use

- Input reads are already cleaned — ViroProfiler's cleaning step is skipped when input reads are already cleaned.
- Data are not shotgun viral metagenomic reads (e.g., amplicon or assembled contigs) — ViroProfiler is designed for analyzing shotgun viral metagenomic data.
- Samples contain excessive environmental contaminations that were not fractionated before sequencing, which could increase running time and result in lower precision; isolate the viral fractions before sequencing instead.

## Inputs

- Raw paired-end FASTQ files (R1/R2) per sample from shotgun viral metagenomic sequencing
- Optional host/contaminant reference index for BBMap decontamination

## Outputs

- Cleaned paired-end FASTQ files (R1/R2) per sample with mates synchronized
- Optional FastQC reports on the cleaned reads

## How to apply

Launch the pipeline's quality-control processes with Nextflow, e.g. 'nextflow run denglab/viroprofiler -profile docker,test', then run FASTP with ViroProfiler defaults '-f 15 -t 1 -F 15 -T 1 --detect_adapter_for_pe -p -n 1 -l 30 -5 -W 4 -M 20 -r -c -g -x': this trims the first/last 15 bases, auto-detects adapters for paired-end data, removes low-quality bases, polyG/polyX tails, and drops reads shorter than 30 bp. Next, optionally run the DECONTAM (BBMap) filter with 'maxindel=1 bwr=0.16 bw=12 quickmatch fast minhits=2 qtrim=rl trimq=10 pigz=True untrim' to quality-trim both ends at Q10 and remove contaminant reads with at least 2 k-mer hits. Optionally run FastQC with '--quiet' on the cleaned reads as a sanity check, then write the cleaned R1/R2 FASTQ per sample keeping mates synchronized. Rationale: removing adapters, low-quality and contaminant reads prevents them from inflating downstream running time and lowering precision, and the cleaning step is skipped when input reads are already cleaned.

## Related tools

- **ViroProfiler** (Nextflow pipeline whose cleaning step performs adapter, low-quality, and contaminant read removal) — https://github.com/deng-lab/viroprofiler
- **FASTP** (Trims first/last 15 bases, auto-detects paired-end adapters, removes low-quality bases, drops reads < 30 bp)
- **BBMap (DECONTAM)** (Quality-trims both ends at Q10 and removes contaminant reads with >= 2 k-mer hits)
- **FastQC** (Optional QC sanity check on cleaned reads run with --quiet)
- **Nextflow** (Workflow engine launching the quality-control processes with the docker,test profile)

## Examples

```
nextflow run denglab/viroprofiler -profile docker,test
```

## Evaluation signals

- The cleaned FASTQ pair (R1/R2) exists for every sample and is in valid FASTQ format.
- R1 and R2 contain equal numbers of reads, confirming mates stayed synchronized.
- Reads shorter than 30 bp are absent, consistent with FASTP's '-l 30' length filter.
- Adapter and first/last 15 bases are trimmed, consistent with '-f 15 -t 1 -F 15 -T 1' and '--detect_adapter_for_pe'.
- Contaminant reads with >= 2 k-mer hits are removed by the BBMap DECONTAM filter (minhits=2).

## Limitations

- The cleaning step is skipped when input reads are already cleaned, so pre-cleaned inputs must not be re-trimmed.
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision; it is recommended to isolate the viral fractions before sequencing.
- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses.

## Evidence

- [methods] Cleaning step description: [evidence span withheld — anchor status: not_found]
- [methods] FASTP defaults: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline availability: "ViroProfiler is freely available at https://github.com/deng-lab/viroprofiler."
- [discussion] Contamination caveat: "excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision."
