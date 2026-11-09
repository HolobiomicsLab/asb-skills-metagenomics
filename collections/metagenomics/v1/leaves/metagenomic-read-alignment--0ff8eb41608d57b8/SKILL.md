---
name: metagenomic-read-alignment
description: Use when shotgun metagenomic FASTQ reads (e.g., the 30-sample BIO-ML
  set from PRJNA544527 or a 121-sample H. pylori eradication cohort) must be mapped
  to a reference database to generate per-sample BAMs for downstream variant detection.
when_to_use_negative:
- Per-sample .filtered.sorted.bam files already exist — skip directly to cellsnp-lite
  pileup (minMAF = 0.02, minCOUNT = 100) and MetaQuad clustering.
- The analysis goal is clustering already-piled variants into shared informative variants
  — that is MetaQuad's role ('metaquad -i cellSNP -o Metaquad --minSample=3'), not
  alignment.
- Input is single-cell mtDNA or scRNA-seq data rather than shotgun metagenomic data
  — the MQuad/cellSNP single-cell lineage, not this metagenomic alignment pipeline,
  applies.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0102
tools:
- name: bwa
  role: Primary aligner (bwa mem) mapping shotgun metagenomic reads to the reference
    database before samtools filtering and sorting.
- name: bowtie2
  role: Alternative alignment tool the MetaQuad pipeline accepts in place of bwa.
- name: samtools
  role: Filters and sorts raw alignments into .filtered.sorted.bam files and computes
    per-sample coverage tables ('samtools coverage ${i}.sorted.bam > ${i}.tab').
- name: sra-tools
  role: Retrieves the source FASTQ files (fasterq-dump) for the BIO-ML samples from
    BioProject PRJNA544527 prior to alignment.
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:5ab1832b103bc4534d8cbf3947613268cf2fd0e550aa713b35b2b793de5873fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  - samtools
  - sra-tools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metagenomic-read-alignment

## Summary

Aligns shotgun metagenomic sequencing reads against a reference database (e.g., a genome set or antibiotic resistance gene (ARG) catalog) with bwa or bowtie2, producing per-sample .filtered.sorted.bam files that serve as the substrate for cellsnp-lite variant pileup and MetaQuad shared informative variant discovery.

## When to use

Use when shotgun metagenomic FASTQ reads (e.g., the 30-sample BIO-ML set from PRJNA544527 or a 121-sample H. pylori eradication cohort) must be mapped to a reference database to generate per-sample BAMs for downstream variant detection.

## When NOT to use

- Per-sample .filtered.sorted.bam files already exist — skip directly to cellsnp-lite pileup (minMAF = 0.02, minCOUNT = 100) and MetaQuad clustering.
- The analysis goal is clustering already-piled variants into shared informative variants — that is MetaQuad's role ('metaquad -i cellSNP -o Metaquad --minSample=3'), not alignment.
- Input is single-cell mtDNA or scRNA-seq data rather than shotgun metagenomic data — the MQuad/cellSNP single-cell lineage, not this metagenomic alignment pipeline, applies.

## Inputs

- Per-sample FASTQ files from shotgun metagenomic sequencing (e.g., 30 BIO-ML samples from BioProject PRJNA544527, or 121 H. pylori eradication therapy samples)
- Reference database sequences (e.g., microbial genomes for BIO-ML, or an antibiotic resistance gene (ARG) reference database)

## Outputs

- Per-sample alignment files named *.filtered.sorted.bam (filtered, coordinate-sorted BAM)
- Per-sample samtools coverage tables (${i}.tab) with genome length and covbases for downstream Genome_coverage.R

## How to apply

Filter and align reads from shotgun metagenomic sequencing to a reference database using bwa mem or bowtie2, then filter and sort the alignments with samtools to produce one .filtered.sorted.bam per sample — this suffix is the MetaQuad pipeline's convention. For the 30 BIO-ML samples, retrieve FASTQs from BioProject PRJNA544527 with sra-tools (fasterq-dump) before alignment; for the ARG study, align the 121 samples to the antibiotic resistance gene reference database. Produce a companion QC table per sample with 'samtools coverage ${i}.sorted.bam > ${i}.tab' to record genome lengths and covbases for the Genome_coverage.R step. The rationale is that MetaQuad's density-based BIC clustering needs consistent, high-quality per-sample alignments so that cellsnp-lite (minMAF = 0.02, minCOUNT = 100) can pile up comparable AD/DP counts across all samples.

## Related tools

- **bwa** (Primary aligner (bwa mem) mapping shotgun metagenomic reads to the reference database before samtools filtering and sorting.)
- **bowtie2** (Alternative alignment tool the MetaQuad pipeline accepts in place of bwa.)
- **samtools** (Filters and sorts raw alignments into .filtered.sorted.bam files and computes per-sample coverage tables ('samtools coverage ${i}.sorted.bam > ${i}.tab').)
- **sra-tools** (Retrieves the source FASTQ files (fasterq-dump) for the BIO-ML samples from BioProject PRJNA544527 prior to alignment.)

## Examples

```
bwa mem ref_db.fa sample_R1.fastq sample_R2.fastq | samtools view -b -F 4 | samtools sort -o sample.filtered.sorted.bam
```

## Evaluation signals

- Every input sample yields a *.filtered.sorted.bam file using the pipeline's recommended suffix, with no sample left unaligned (30 for BIO-ML, 121 for the ARG dataset).
- Running 'samtools coverage ${i}.sorted.bam > ${i}.tab' succeeds per sample and the resulting .tab files expose genome length and covbases, matching the article's Genome_coverage.R workflow.
- The aligned BAMs feed cellsnp-lite pileup without errors, and on the BIO-ML data the full pipeline reproduces 67,241 variants (64,682 with >=2 clusters) — a failed or truncated alignment breaks this count.
- On the ARG dataset the aligned BAMs support reproducing 7591 informative variants across 529 ARGs within the reported ~60-minute runtime.

## Limitations

- Detection is bounded by the reference database: only reads mapping to supplied genomes or ARG genes contribute to variant pileup, so alignment coverage of each sample (covbases) directly gates which loci cellsnp-lite and MetaQuad can interrogate.
- cellsnp-lite (the downstream consumer of these alignments) is noted as a faster version of cellSNP that is 'still at testing stage so might be unstable', and a higher minCOUNT is recommended to reduce computation time for subsequent steps.

## Evidence

- [intro] Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2.: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2."
- [methods] samtools coverage ${i}.sorted.bam > ${i}.tab: "samtools coverage ${i}.sorted.bam > ${i}.tab"
- [readme] The recommended suffix for the final files is ".filtered.sorted.bam".: "The recommended suffix for the final files is ".filtered.sorted.bam"."
- [readme] A higher value is recommended to reduce computation time for subsequent steps.: "A higher value is recommended to reduce computation time for subsequent steps."
