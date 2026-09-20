---
name: sra-metagenomic-data-retrieval
description: Use when a downstream variant-discovery or strain-analysis pipeline (e.g.
  cellsnp-lite + MetaQuad) needs raw sequencing reads from a publicly archived metagenomic
  study and only the BioProject accession is known.
when_to_use_negative:
- When the raw reads are already available locally as FASTQ or BAM files — go straight
  to bwa/samtools alignment instead.
- When the input is single-cell sequencing data rather than shotgun metagenomic samples
  — the MetaQuad BIO-ML workflow is tailored to shotgun metagenomic data.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3697
tools:
- name: sra-tools (fasterq-dump)
  role: Downloads raw sequencing reads from the SRA entry of BioProject PRJNA544527
    as FASTQ files for each of the 30 samples
- name: bwa
  role: 'Downstream consumer: aligns the retrieved FASTQ reads to the reference database
    to produce .filtered.sorted.bam files'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:ee6dc9d77b3e7c1dc6df652f92b514e3bc296a9a17695b692410e81ca9e9eb18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaQuad
  - Python
  - bwa
  - cellsnp-lite
  - sra-tools (fasterq-dump)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# sra-metagenomic-data-retrieval

## Summary

Retrieve raw shotgun metagenomic FASTQ files from a public BioProject on SRA/NCBI (e.g. the 30-sample BIO-ML dataset from BioProject PRJNA544527) using sra-tools' fasterq-dump, as the entry step of a MetaQuad shared-informative-variant pipeline.

## When to use

Use when a downstream variant-discovery or strain-analysis pipeline (e.g. cellsnp-lite + MetaQuad) needs raw sequencing reads from a publicly archived metagenomic study and only the BioProject accession is known.

## When NOT to use

- When the raw reads are already available locally as FASTQ or BAM files — go straight to bwa/samtools alignment instead.
- When the input is single-cell sequencing data rather than shotgun metagenomic samples — the MetaQuad BIO-ML workflow is tailored to shotgun metagenomic data.

## Inputs

- BioProject accession (e.g. PRJNA544527) for a shotgun metagenomic study
- SRA run accessions for each of the 30 metagenomic samples

## Outputs

- 30 raw FASTQ files, one per shotgun metagenomic sample, ready for bwa mem alignment

## How to apply

Identify the BioProject accession for the target study (for the MetaQuad reproduction it is PRJNA544527, containing 30 shotgun metagenomic samples from 3 healthy individuals). Use sra-tools, specifically fasterq-dump, to download each sample's raw reads, producing one FASTQ file per sample. These FASTQ files are then aligned to a reference database with bwa mem and filtered/sorted with samtools into .filtered.sorted.bam files, so correct retrieval must yield all 30 samples with sufficient read depth for downstream minCOUNT=100 variant pile-up. Verify retrieval by checking the number of downloaded FASTQ files matches the expected sample count and that alignment (e.g. samtools coverage) shows adequate covered bases per sample.

## Related tools

- **sra-tools (fasterq-dump)** (Downloads raw sequencing reads from the SRA entry of BioProject PRJNA544527 as FASTQ files for each of the 30 samples)
- **bwa** (Downstream consumer: aligns the retrieved FASTQ reads to the reference database to produce .filtered.sorted.bam files)

## Examples

```
fasterq-dump --split-files <SRR_accession>  # repeat for each of the 30 runs in BioProject PRJNA544527
```

## Evaluation signals

- Number of downloaded FASTQ files equals the expected sample count (30 for PRJNA544527).
- samtools coverage on the resulting .filtered.sorted.bam files shows non-trivial covered bases for every sample.
- Downstream cellsnp-lite pile-up (minMAF=0.02, minCOUNT=100) recovers 67,241 variants on the BIO-ML dataset, confirming reads were retrieved intact.
- Alignment step completes for all samples with valid .filtered.sorted.bam output (no empty or truncated BAM files).

## Limitations

- Only covers retrieval of publicly archived data; if a BioProject lacks run accessions or the SRA entries are unavailable, the pipeline cannot start.
- Retrieval alone does not guarantee sufficient sequencing depth — cellsnp-lite thresholds of minMAF=0.02 and minCOUNT=100 require adequate per-sample coverage.

## Evidence

- [results] Retrieve the 30 FASTQ files of shotgun metagenomic samples from BioProject PRJNA544527 using sra-tools (fasterq-dump).: [evidence span withheld — anchor status: not_found]
- [intro] Filter and align reads from shotgun metagenomic sequencing to a reference database using bwa or bowtie2, producing .filtered.sorted.bam files: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2."
- [readme] Pile up variants using cellsnp-lite with recommended parameters minMAF = 0.02 and minCOUNT = 100: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
