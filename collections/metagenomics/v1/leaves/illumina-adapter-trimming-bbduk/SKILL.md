---
name: illumina-adapter-trimming-bbduk
description: Use when raw paired-end or single-end Illumina FASTQ reads (fastq/fastq.gz)
  enter a metagenomic pipeline and must be stripped of Illumina adapters and low-quality
  bases before SSU rRNA filtering or assembly.
when_to_use_negative:
- Input reads are long-read (PacBio/ONT) data — the workflow and parameters target
  short Illumina shotgun reads
- Reads are already adapter-trimmed and quality-filtered (e.g. input is the trimmed
  16S/18S output of a prior bbduk step)
- The goal is SSU read extraction or taxonomic classification rather than quality
  control — use SortMeRNA or the QIIME2 sklearn classifier instead
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0080
tools:
- name: BBTools
  role: Provides bbduk.sh for adapter removal and quality trimming of Illumina reads
    (ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1, minlength=60)
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: FastQC
  role: Quality reporting on input files (before_fastqc) and trimmed outputs (after_fastqc)
    as .html reports
  repo: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
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
  source_content_hash: sha256:885de73b303eb0c515b22e844d708e33d20f560e193238e2aefbcd230bb2672c
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
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# illumina-adapter-trimming-bbduk

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Adapter removal and quality trimming of raw Illumina (meta)genomic reads with bbduk.sh from BBTools, as implemented in the RiboTaxa quality-control stage prior to SSU rRNA read extraction and reconstruction.

## When to use

Use when raw paired-end or single-end Illumina FASTQ reads (fastq/fastq.gz) enter a metagenomic pipeline and must be stripped of Illumina adapters and low-quality bases before SSU rRNA filtering or assembly.

## When NOT to use

- Input reads are long-read (PacBio/ONT) data — the workflow and parameters target short Illumina shotgun reads
- Reads are already adapter-trimmed and quality-filtered (e.g. input is the trimmed 16S/18S output of a prior bbduk step)
- The goal is SSU read extraction or taxonomic classification rather than quality control — use SortMeRNA or the QIIME2 sklearn classifier instead

## Inputs

- Raw paired-end Illumina metagenomic reads in FASTQ or fastq.gz format (loaded from DATA_DIR)
- Illumina adapter sequences (known Illumina adapters/artifacts supplied to bbduk.sh)

## Outputs

- Adapter-trimmed and quality-trimmed paired FASTQ files in the quality_control directory
- Per-file FastQC .html reports (before_fastqc and after_fastqc)
- Aggregated MultiQC report summarizing all (meta)genomics files

## How to apply

Load the raw paired-end metagenomic reads from the configured DATA_DIR and run bbduk.sh from BBTools (v37.62) with ktrim=r (right-side adapter trimming), kmer=21 (adapter k-mer length), qtrim=rl (trim both ends), trimq=20 (quality threshold), maxns=1 (maximum Ns after trimming), and minlength=60 (minimum post-trimming read length). Reads shorter than 60 bp, with more than 1 N, or containing regions below Q20 are trimmed or discarded. Run FastQC on the input files (before_fastqc) and the trimmed output (after_fastqc) to generate .html quality reports, then aggregate all per-file reports with MultiQC into a single overview. Validate by confirming the trimmed FASTQ pair exists in the quality_control directory and that FastQC/MultiQC reports were generated before passing reads to SortMeRNA-based 16S/18S filtering.

## Related tools

- **BBTools** (Provides bbduk.sh for adapter removal and quality trimming of Illumina reads (ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1, minlength=60)) — https://jgi.doe.gov/data-and-tools/bbtools/
- **FastQC** (Quality reporting on input files (before_fastqc) and trimmed outputs (after_fastqc) as .html reports) — http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **MultiQC** (Aggregates per-file FastQC reports into a single multiqc overview of all (meta)genomics files) — https://github.com/MultiQC/MultiQC

## Examples

```
bbduk.sh in=reads_R1.fastq.gz in2=reads_R2.fastq.gz out=trimmed_R1.fastq.gz out2=trimmed_R2.fastq.gz ktrim=r k=21 qtrim=rl trimq=20 maxns=1 minlength=60
```

## Evaluation signals

- The trimmed FASTQ pair exists in the quality_control directory after the bbduk.sh run
- FastQC reports were generated for both input (before_fastqc) and output (after_fastqc) files, and MultiQC produced a single aggregated report
- No reads in the trimmed output are shorter than 60 bp (minlength=60) or contain more than 1 N (maxns=1)
- Adapter k-mers (k-mer 21, right-side ktrim=r) are absent from the after_fastqc adapter-content plots
- Per-base quality in after_fastqc reports shows both ends trimmed to at least Q20 (qtrim=rl, trimq=20)

## Limitations

- Reads shorter than 60 bp or with more than 1 N are discarded rather than repaired, so heavily degraded libraries may lose a large fraction of reads
- Trimming reduces read length, which can affect the yield of downstream SSU read extraction and full-length rRNA reconstruction in RiboTaxa
- The pipeline was designed and validated for Illumina metagenomic sequences; parameters (kmer=21, trimq=20) may not transfer directly to other platforms or read types

## Evidence

- [methods] Quality control using BBTOOLS ... Trim reads to remove bases matching adapter sequences ... Reads shorter than this length (bases) after trimming will be discarded: [evidence span withheld — anchor status: not_found]
