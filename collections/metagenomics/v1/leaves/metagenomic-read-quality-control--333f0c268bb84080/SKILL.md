---
name: metagenomic-read-quality-control
description: Use when raw paired-end or single-end Illumina metagenomic FASTQ/fastq.gz
  reads must be adapter-trimmed and quality-trimmed before SSU rRNA read extraction
  or any 16S/18S-based taxonomic classification.
when_to_use_negative:
- Input reads have already been adapter- and quality-trimmed (re-trimming with minlength=60/trimq=20
  would needlessly discard data)
- Data are amplicon-only reads destined for direct QIIME2 classification rather than
  RiboTaxa shotgun-based SSU extraction
- Input is a database or reference sequence (e.g. SILVA SSU 138.1 FASTA) rather than
  raw sequencing reads — use database indexing instead
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BBTools
  role: Adapter removal and quality trimming via bbduk.sh (ktrim=r, kmer=21, qtrim=rl,
    trimq=20, maxns=1, minlength=60)
  repo: https://github.com/oschakoory/RiboTaxa
- name: FastQC
  role: Per-file read quality reports on raw inputs (before_fastqc) and trimmed outputs
    (after_fastqc)
  repo: https://github.com/oschakoory/RiboTaxa
- name: MultiQC
  role: Aggregates per-file FastQC reports into a single overview of all (meta)genomics
    files
  repo: https://github.com/MultiQC/MultiQC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:db81a15f73104b51d6dc92f33ff3d7c5d6347938aaad7ddaf98b2b44f28df15b
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metagenomic-read-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control stage of the RiboTaxa pipeline: removes Illumina adapters and low-quality bases from raw Illumina (meta)genomic FASTQ reads using bbduk.sh (BBTools), then verifies results with FastQC and MultiQC reports before SSU rRNA extraction.

## When to use

Use when raw paired-end or single-end Illumina metagenomic FASTQ/fastq.gz reads must be adapter-trimmed and quality-trimmed before SSU rRNA read extraction or any 16S/18S-based taxonomic classification.

## When NOT to use

- Input reads have already been adapter- and quality-trimmed (re-trimming with minlength=60/trimq=20 would needlessly discard data)
- Data are amplicon-only reads destined for direct QIIME2 classification rather than RiboTaxa shotgun-based SSU extraction
- Input is a database or reference sequence (e.g. SILVA SSU 138.1 FASTA) rather than raw sequencing reads — use database indexing instead

## Inputs

- Raw single-end or paired-end Illumina metagenomic reads in FASTQ (.fastq) or compressed (.fastq.gz) format

## Outputs

- Adapter- and quality-trimmed paired FASTQ files in the quality_control directory
- FastQC .html reports per input and output file (before_fastqc, after_fastqc)
- Aggregated MultiQC report summarizing all (meta)genomics files

## How to apply

Load the raw single-end or paired-end reads (fastq or compressed fastq.gz) from the configured DATA_DIR and run bbduk.sh from BBTools (v37.62) with ktrim=r (right-side adapter trimming), kmer=21 (adapter k-mer length), qtrim=rl (trim both ends), trimq=20 (Q20 quality threshold), maxns=1 (maximum Ns after trimming), and minlength=60 so reads shorter than 60 bp after trimming are discarded. Run FastQC on the input files (before_fastqc) and on the trimmed outputs (after_fastqc) to generate .html quality reports, then aggregate the per-file reports with MultiQC into a single overview of all (meta)genomics files. The rationale is that RiboTaxa's downstream steps (SortMeRNA 16S/18S extraction, EMIRGE/MetaRib SSU reconstruction) need clean reads free of adapters and low-quality tails; the paper explicitly supports read trimming as a pipeline capability. The RiboTaxa README confirms BBTOOLS is the tool designated 'for adapters removal and trimming', with FastQC/MultiQC for quality control.

## Related tools

- **BBTools** (Adapter removal and quality trimming via bbduk.sh (ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1, minlength=60)) — https://github.com/oschakoory/RiboTaxa
- **FastQC** (Per-file read quality reports on raw inputs (before_fastqc) and trimmed outputs (after_fastqc)) — https://github.com/oschakoory/RiboTaxa
- **MultiQC** (Aggregates per-file FastQC reports into a single overview of all (meta)genomics files) — https://github.com/MultiQC/MultiQC

## Examples

```
bbduk.sh in=reads_R1.fq in2=reads_R2.fq out=trimmed_R1.fq out2=trimmed_R2.fq ktrim=r kmer=21 qtrim=rl trimq=20 maxns=1 minlength=60
```

## Evaluation signals

- The trimmed FASTQ pair exists in the quality_control directory after the bbduk.sh run
- FastQC .html reports were generated for both before_fastqc and after_fastqc directories
- A MultiQC aggregate report covering all (meta)genomics input files was produced
- No output read is shorter than 60 bp or contains more than 1 N — reads failing these constraints must have been trimmed or discarded per the bbduk parameters
- After-fastqc report shows adapter content and low-quality (below Q20) base regions removed relative to the before-fastqc report

## Limitations

- Reads shorter than 60 bp after trimming are discarded entirely, which may remove very short but genuine rRNA fragments
- The Q20 two-end trim and maxns=1 settings are strict; heavily degraded libraries may lose a large fraction of reads
- Chimera detection was not included downstream in the 16S rRNA gene reconstruction workflow, so QC does not address chimeric sequences

## Evidence

- [methods] RiboTaxa's quality-control stage processes Illumina shotgun reads with bbduk.sh (BBTools v37.62, k-mer = 21) to remove Illumina adapters and known Illumina artifacts and to quality-trim both ends to Q20.: [evidence span withheld — anchor status: not_found]
