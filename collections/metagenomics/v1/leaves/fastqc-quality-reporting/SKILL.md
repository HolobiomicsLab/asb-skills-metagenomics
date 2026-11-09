---
name: fastqc-quality-reporting
description: Use when raw Illumina metagenomic FASTQ/fastq.gz reads enter a QC stage
  and per-file FastQC reports plus a consolidated MultiQC summary are needed before
  and after adapter/quality trimming with bbduk.
when_to_use_negative:
- Input is already trimmed, filtered SSU rRNA reads, or assembled SSU sequences —
  FastQC bracketing of bbduk trimming no longer applies and the sortmerna filtering
  step should be used instead
- The goal is taxonomic classification of reconstructed SSU rRNA sequences (QIIME
  2 sklearn classifier stage), not read-level quality assessment
- Working with non-Illumina read types where Illumina adapter/artifact assumptions
  of the RiboTaxa QC stage do not hold
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3941
tools:
- name: FastQC
  role: Produces per-file HTML quality reports on input files (before_fastqc) and
    trimmed output files (after_fastqc) within the RiboTaxa quality-control stage
  repo: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- name: MultiQC
  role: Aggregates the per-file FastQC reports into a single multiqc overview of all
    (meta)genomics files
  repo: MultiQC/MultiQC
- name: BBTools
  role: Upstream trimming tool (bbduk.sh, ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1,
    minlength=60) whose output the after-trim FastQC reports evaluate
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:2364c1f39c979620a8fe646cb1e13877db29e93e819b5b9d1d0861402bfdee92
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - FastQC
  - MultiQC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fastqc-quality-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates FastQC HTML quality reports on raw and bbduk-trimmed Illumina (meta)genomic FASTQ files and consolidates them with MultiQC, providing the quality-control evidence layer of the RiboTaxa pipeline before SSU rRNA extraction.

## When to use

Use when raw Illumina metagenomic FASTQ/fastq.gz reads enter a QC stage and per-file FastQC reports plus a consolidated MultiQC summary are needed before and after adapter/quality trimming with bbduk.

## When NOT to use

- Input is already trimmed, filtered SSU rRNA reads, or assembled SSU sequences — FastQC bracketing of bbduk trimming no longer applies and the sortmerna filtering step should be used instead
- The goal is taxonomic classification of reconstructed SSU rRNA sequences (QIIME 2 sklearn classifier stage), not read-level quality assessment
- Working with non-Illumina read types where Illumina adapter/artifact assumptions of the RiboTaxa QC stage do not hold

## Inputs

- Raw single-end or paired-end Illumina (meta)genomic reads in FASTQ or compressed fastq.gz format
- bbduk.sh-trimmed FASTQ pair produced in the quality_control directory

## Outputs

- Per-file FastQC HTML quality reports (before_fastqc on raw reads, after_fastqc on trimmed reads)
- A single MultiQC report aggregating all per-file FastQC reports across samples

## How to apply

Run FastQC on the raw input FASTQ files (before_fastqc) and again on the bbduk.sh-trimmed outputs (after_fastqc) so per-file HTML reports bracket the trimming step, which uses BBTools with ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1 and minlength=60. Aggregate all per-file FastQC reports with MultiQC into a single report that summarizes all (meta)genomics files, since FastQC itself only reports per file and RiboTaxa pairs the two tools specifically 'for quality control'. Interpret the reports comparatively: the after-trim report should show that reads shorter than 60 bp, reads with more than 1 N, and regions below Q20 have been trimmed or discarded per the bbduk parameters. Rationale: direct classification of short Illumina reads yields high false-positive rates, so this QC gate precedes SortMeRNA-based 16S/18S filtering and SSU reconstruction in RiboTaxa.

## Related tools

- **FastQC** (Produces per-file HTML quality reports on input files (before_fastqc) and trimmed output files (after_fastqc) within the RiboTaxa quality-control stage) — http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **MultiQC** (Aggregates the per-file FastQC reports into a single multiqc overview of all (meta)genomics files) — MultiQC/MultiQC
- **BBTools** (Upstream trimming tool (bbduk.sh, ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1, minlength=60) whose output the after-trim FastQC reports evaluate) — https://jgi.doe.gov/data-and-tools/bbtools/

## Examples

```
bbduk.sh in=R1.fq.gz in2=R2.fq.gz out=tR1.fq.gz out2=tR2.fq.gz ktrim=r kmer=21 qtrim=rl trimq=20 maxns=1 minlength=60; fastqc R1.fq.gz tR1.fq.gz -o after_fastqc; multiqc .
```

## Evaluation signals

- Both before_fastqc and after_fastqc HTML reports exist for every input FASTQ file in the quality_control directory
- A MultiQC report consolidates all per-file FastQC reports into one overview across samples
- After-trim FastQC/MultiQC profiles show adapter content removed and reads with regions below Q20, more than 1 N, or shorter than 60 bp trimmed or discarded per the bbduk parameters
- The trimmed FASTQ pair is present in the quality_control directory before proceeding to SortMeRNA 16S/18S filtering
- Per-base and per-sequence quality distributions improve in the after_fastqc report relative to before_fastqc

## Limitations

- FastQC/MultiQC reports are descriptive only — quality problems must be remediated by the upstream bbduk.sh trimming call, not by the reports themselves
- FastQC produces one report per file, so multi-sample assessment depends entirely on the MultiQC aggregation step
- Reads shorter than 60 bp after trimming are discarded (minlength=60), so heavily degraded libraries can lose data before SSU reconstruction

## Evidence

- [readme] For quality control :FastQC, MultiQC: [evidence span withheld — anchor status: not_found]
- [readme] Aggregate bioinformatics results across many samples into a single report: [evidence span withheld — anchor status: external_doc]
