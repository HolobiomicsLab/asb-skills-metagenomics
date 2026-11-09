---
name: short-read-quality-trimming
description: Use when raw Illumina paired-end metagenomic FASTQ/fastq.gz reads must
  be stripped of Illumina adapters and low-quality bases before downstream SSU rRNA
  filtering, assembly, or classification.
when_to_use_negative:
- Input reads are long-read (PacBio/ONT) data — bbduk adapter trimming with kmer=21
  targets short Illumina reads only
- Reads are already trimmed FASTQ sequences, e.g. the filtered 16S/18S output of SortMeRNA
  ('This folder contains filtered 16S/18S sequences from your trimmed (meta)genomics
  sequence files')
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: BBTools
  role: bbduk.sh performs adapter removal (ktrim=r, kmer=21) and quality trimming
    (qtrim=rl, trimq=20, maxns=1, minlength=60)
  repo: https://github.com/oschakoory/RiboTaxa
- name: FastQC
  role: Per-file quality reporting before and after trimming (before_fastqc / after_fastqc)
  repo: https://github.com/oschakoory/RiboTaxa
- name: MultiQC
  role: Aggregates per-file FastQC reports into a single multiqc overview of all (meta)genomics
    files
  repo: https://github.com/MultiQC/MultiQC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:09b9902ce85082d732bf6f152976a7e9140d978dee3bcbdc253021ee92189319
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - FastQC
  - MultiQC
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# short-read-quality-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adapter removal and quality trimming of raw Illumina (meta)genomic FASTQ reads using bbduk.sh from BBTools, as performed in the RiboTaxa pipeline's quality-control stage prior to SSU rRNA read extraction and reconstruction.

## When to use

Use when raw Illumina paired-end metagenomic FASTQ/fastq.gz reads must be stripped of Illumina adapters and low-quality bases before downstream SSU rRNA filtering, assembly, or classification.

## When NOT to use

- Input reads are long-read (PacBio/ONT) data — bbduk adapter trimming with kmer=21 targets short Illumina reads only
- Reads are already trimmed FASTQ sequences, e.g. the filtered 16S/18S output of SortMeRNA ('This folder contains filtered 16S/18S sequences from your trimmed (meta)genomics sequence files')

## Inputs

- Raw paired-end Illumina metagenomic reads in FASTQ or fastq.gz format from DATA_DIR
- Illumina adapter/artifact sequences (known Illumina artifacts used by bbduk)

## Outputs

- Trimmed FASTQ read pairs in the quality_control directory
- FastQC .html reports (before_fastqc and after_fastqc)
- Aggregated MultiQC report covering all (meta)genomics files

## How to apply

Load raw paired-end metagenomic reads (fastq/fastq.gz) from the configured DATA_DIR and run bbduk.sh (BBTools v37.62) with ktrim=r for right-side adapter trimming, kmer=21 for adapter k-mer length, qtrim=rl to trim both ends, trimq=20 as the quality threshold, maxns=1 for the maximum allowed Ns, and minlength=60 so reads shorter than 60 bp after trimming are discarded. Run FastQC on the input files (before_fastqc) and on the trimmed outputs (after_fastqc) to generate .html quality reports, then aggregate the per-file reports with MultiQC into a single overview. Rationale: removing adapters and bases below Q20 prevents low-quality sequence from contaminating SSU read extraction (SortMeRNA) and full-length SSU rRNA reconstruction (EMIRGE/MetaRib), which the article shows is critical because direct classification of short reads yields high false-positive rates.

## Related tools

- **BBTools** (bbduk.sh performs adapter removal (ktrim=r, kmer=21) and quality trimming (qtrim=rl, trimq=20, maxns=1, minlength=60)) — https://github.com/oschakoory/RiboTaxa
- **FastQC** (Per-file quality reporting before and after trimming (before_fastqc / after_fastqc)) — https://github.com/oschakoory/RiboTaxa
- **MultiQC** (Aggregates per-file FastQC reports into a single multiqc overview of all (meta)genomics files) — https://github.com/MultiQC/MultiQC

## Examples

```
bbduk.sh in=sample_R1.fastq.gz in2=sample_R2.fastq.gz out=trimmed_R1.fastq.gz out2=trimmed_R2.fastq.gz ktrim=r kmer=21 qtrim=rl trimq=20 maxns=1 minlength=60
```

## Evaluation signals

- The trimmed FASTQ pair exists in the quality_control directory
- FastQC reports were generated for both input (before_fastqc) and trimmed output (after_fastqc) files
- A single aggregated MultiQC overview of all (meta)genomics files was produced
- Reads shorter than 60 bp, with more than 1 N, or with regions below Q20 were trimmed or discarded per the bbduk parameters
- after_fastqc reports show improved per-base quality and absence of adapter content relative to before_fastqc

## Limitations

- Trimming discards reads shorter than 60 bp and reads with more than 1 N, so heavily degraded libraries lose material before SSU extraction
- Quality trimming shortens reads, which can reduce the sortable/alignable length available for rRNA-specialized tools like SortMeRNA

## Evidence

- [methods] Quality control using BBTOOLS ... Trim reads to remove bases matching adapter sequences ... Reads shorter than this length (bases) after trimming will be discarded: [evidence span withheld — anchor status: not_found]
- [results] RiboTaxa's quality-control stage processes Illumina shotgun reads with bbduk.sh (BBTools v37.62, k-mer = 21) to remove Illumina adapters and known Illumina artifacts and to quality-trim both ends to Q20.: [evidence span withheld — anchor status: not_found]
