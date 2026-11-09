---
name: shotgun-metagenomic-pileup-processing
description: Use when you have .filtered.sorted.bam files from shotgun metagenomic
  sequencing and need an allele-count (SNP x sample) pileup matrix to feed variant
  discovery such as MetaQuad.
when_to_use_negative:
- Reads are not yet filtered and aligned — the pipeline requires .filtered.sorted.bam
  from bwa or bowtie2 as upstream input, not raw FASTQ
- Data are single-cell mitochondrial/mtDNA data intended for MQuad's original scope
  rather than shotgun metagenomic samples (MQuad/CellSNP serve that lineage; MetaQuad
  is the metagenomic extension)
- The goal is bulk per-sample variant calling rather than detecting shared informative
  variants across a population of metagenomic samples
edam_operation: http://edamontology.org/operation_3227
edam_topics: []
tools:
- name: cellsnp-lite
  role: Performs the SNP pileup over the aligned shotgun metagenomic BAMs to generate
    the cellSNP output folder with allele count matrices
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MetaQuad
  role: Downstream consumer of the cellSNP output folder that applies density-based
    clustering to detect shared informative variants
  repo: https://github.com/holab-hku/MetaQuad
- name: bwa
  role: Upstream alignment of filtered shotgun metagenomic reads to a reference database
- name: bowtie2
  role: Alternative upstream alignment tool for producing .filtered.sorted.bam files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:3b94f3880c8583545410e498dab59e083a1c59a72583e43b1373ef9c4d1fb75b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# shotgun-metagenomic-pileup-processing

## Summary

Pile up SNPs from aligned shotgun metagenomic samples using cellsnp-lite (minMAF = 0.02, minCOUNT = 100) to produce a cellSNP output folder that downstream tools like MetaQuad consume to discover shared informative variants.

## When to use

Use when you have .filtered.sorted.bam files from shotgun metagenomic sequencing and need an allele-count (SNP x sample) pileup matrix to feed variant discovery such as MetaQuad.

## When NOT to use

- Reads are not yet filtered and aligned — the pipeline requires .filtered.sorted.bam from bwa or bowtie2 as upstream input, not raw FASTQ
- Data are single-cell mitochondrial/mtDNA data intended for MQuad's original scope rather than shotgun metagenomic samples (MQuad/CellSNP serve that lineage; MetaQuad is the metagenomic extension)
- The goal is bulk per-sample variant calling rather than detecting shared informative variants across a population of metagenomic samples

## Inputs

- .filtered.sorted.bam files from bwa or bowtie2 alignment of shotgun metagenomic reads
- Reference database FASTA used for alignment
- cellsnp-lite parameters minMAF = 0.02 and minCOUNT = 100

## Outputs

- cellSNP output folder with AD and DP sparse count matrices (SNP x sample)
- cellSNP.cells.vcf.gz pileup VCF consumable by MetaQuad

## How to apply

After filtering and aligning reads to a reference database with bwa or bowtie2 (producing files with the recommended suffix '.filtered.sorted.bam'), run cellsnp-lite on the BAM set to pile up bi-allelic SNPs into allele read-count matrices. Apply the recommended parameters minMAF = 0.02 and minCOUNT = 100; the README notes a higher value can be used to reduce computation time for subsequent steps. The result is a cellSNP output folder containing AD and DP sparse matrices plus a VCF, which MetaQuad takes directly as input (e.g. 'metaquad -i cellSNP -o Metaquad --minSample=5'). Correct application is confirmed when MetaQuad accepts the folder unchanged and the subsequent variant detection runs with the recommended minSample (5% of total sample size) and num_of_clusters >= 2 filters.

## Related tools

- **cellsnp-lite** (Performs the SNP pileup over the aligned shotgun metagenomic BAMs to generate the cellSNP output folder with allele count matrices) — https://github.com/single-cell-genetics/cellsnp-lite
- **MetaQuad** (Downstream consumer of the cellSNP output folder that applies density-based clustering to detect shared informative variants) — https://github.com/holab-hku/MetaQuad
- **bwa** (Upstream alignment of filtered shotgun metagenomic reads to a reference database)
- **bowtie2** (Alternative upstream alignment tool for producing .filtered.sorted.bam files)

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- cellsnp-lite ran with the recommended thresholds minMAF = 0.02 and minCOUNT = 100
- Output folder contains the cellSNP structure (AD and DP sparse matrices, VCF) that MetaQuad accepts via -i
- Downstream MetaQuad run succeeds on the folder, e.g. 'metaquad -i cellSNP -o Metaquad --minSample=5', with minSample set to 5% of the total sample size and variants accepted with num_of_clusters >= 2
- In benchmarking (n=100 simulations), matched variants against simulated ground truth reproduce the pattern that F1 trends toward 1 at high mean depth while false positives are reduced relative to BCFtools, GATK HaplotypeCaller, VarScan2, MetaSNV, and inStrain

## Limitations

- A higher minCOUNT/minMAF is recommended to reduce computation time for subsequent steps, so the defaults trade sensitivity for speed
- cellsnp-lite README notes it is a faster version of cellSNP that might still be unstable at the testing stage (per the MQuad README context)
- Pileup quality depends on alignment suffix/format conventions: final aligned files should use '.filtered.sorted.bam'

## Evidence

- [intro] Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [intro] Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2.: "Filter and align reads from shotgun metagenomic sequencing to a reference database using alignment tools such as bwa or bowtie2."
- [methods] MetaQuad takes the output folder of cellSNP-lite: "MetaQuad takes the output folder of cellSNP-lite"
- [readme] A higher value is recommended to reduce computation time for subsequent steps.: "with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100. A higher value is recommended to reduce computation time for subsequent steps."
- [readme] The recommended suffix for the final files is ".filtered.sorted.bam".: "The recommended suffix for the final files is ".filtered.sorted.bam"."
