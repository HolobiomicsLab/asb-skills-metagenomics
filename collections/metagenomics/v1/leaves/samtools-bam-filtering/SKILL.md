---
name: samtools-bam-filtering
description: Use when raw FASTQ reads from shotgun metagenomic samples must be aligned
  to a reference database and sorted/filtered into .filtered.sorted.bam files before
  variant pileup with cellsnp-lite and MetaQuad.
when_to_use_negative:
- Input is already aligned — if .filtered.sorted.bam files exist, skip directly to
  cellsnp-lite pileup
- Data are single-cell mtDNA reads processed with the cellSNP/MQuad single-cell pipeline
  rather than bulk shotgun metagenomic samples
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bwa
  role: Aligns filtered shotgun metagenomic reads to the reference database (bwa mem)
  repo: https://github.com/holab-hku/MetaQuad.git
- name: bowtie2
  role: Alternative alignment tool mentioned by the article for producing the filtered
    sorted BAM files
- name: samtools
  role: Filters and sorts bwa alignments into .filtered.sorted.bam and computes coverage
    statistics ('samtools coverage ${i}.sorted.bam > ${i}.tab')
  repo: https://github.com/holab-hku/MetaQuad.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:b7336056e6bad10891de2b2478c0ef3d7dffdb3975e3fc09e41496bb66c2f5f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad.git
  tools:
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  - samtools
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# samtools-bam-filtering

## Summary

Filter and align shotgun metagenomic reads to a reference database with bwa (or bowtie2) and samtools to produce per-sample .filtered.sorted.bam files, the required input format for the MetaQuad shared informative variants pipeline.

## When to use

Use when raw FASTQ reads from shotgun metagenomic samples must be aligned to a reference database and sorted/filtered into .filtered.sorted.bam files before variant pileup with cellsnp-lite and MetaQuad.

## When NOT to use

- Input is already aligned — if .filtered.sorted.bam files exist, skip directly to cellsnp-lite pileup
- Data are single-cell mtDNA reads processed with the cellSNP/MQuad single-cell pipeline rather than bulk shotgun metagenomic samples

## Inputs

- Raw shotgun metagenomic sequencing reads (FASTQ files, e.g., from SRA via fasterq-dump)
- Reference database (reference genome/metagenome in FASTA format for bwa indexing)

## Outputs

- Per-sample filtered and sorted alignment files (.filtered.sorted.bam)
- Optional per-sample coverage tables (${i}.tab from samtools coverage)

## How to apply

Retrieve raw reads (e.g., via fasterq-dump from SRA accessions such as BioProject PRJNA544527) and align them to the reference database using 'bwa mem'. Filter and sort the resulting alignments with samtools so each sample yields a .filtered.sorted.bam file; the MetaQuad README explicitly recommends '.filtered.sorted.bam' as the suffix for the final files. Optionally compute per-sample coverage statistics with 'samtools coverage ${i}.sorted.bam > ${i}.tab' as done in the article's methods for genome coverage assessment. These BAM files then feed cellsnp-lite pileup (minMAF = 0.02, minCOUNT = 100) and downstream MetaQuad density-based BIC clustering.

## Related tools

- **bwa** (Aligns filtered shotgun metagenomic reads to the reference database (bwa mem)) — https://github.com/holab-hku/MetaQuad.git
- **bowtie2** (Alternative alignment tool mentioned by the article for producing the filtered sorted BAM files)
- **samtools** (Filters and sorts bwa alignments into .filtered.sorted.bam and computes coverage statistics ('samtools coverage ${i}.sorted.bam > ${i}.tab')) — https://github.com/holab-hku/MetaQuad.git

## Examples

```
bwa mem reference.fa sample_R1.fq sample_R2.fq | samtools view -b -F 4 - | samtools sort -o sample.filtered.sorted.bam
```

## Evaluation signals

- Each sample yields a .filtered.sorted.bam file with the exact recommended suffix expected by the MetaQuad pipeline
- BAM files are coordinate-sorted and pass samtools quickcheck without errors
- samtools coverage reports non-zero covbases for samples, confirming reads aligned to the reference database
- Downstream cellsnp-lite run on the BAMs reproduces the expected pileup (e.g., 67,241 variants on the 30-sample BIO-ML dataset)

## Limitations

- Higher cellsnp-lite minCOUNT values are recommended to reduce computation time, implying stringent upstream alignment filtering alone may leave many low-count sites
- The article notes cellsnp-lite is 'a faster version of cellSNP, still at testing stage so might be unstable' per the MQuad README, so upstream BAM quality directly affects pileup reliability

## Evidence

- [intro] Filter and align reads from shotgun metagenomic sequencing to a reference database using bwa or bowtie2, producing .filtered.sorted.bam files: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2."
- [readme] The recommended suffix for the final files is '.filtered.sorted.bam'.: "The recommended suffix for the final files is ".filtered.sorted.bam"."
- [methods] samtools coverage command in the methods: "samtools coverage ${i}.sorted.bam > ${i}.tab"
