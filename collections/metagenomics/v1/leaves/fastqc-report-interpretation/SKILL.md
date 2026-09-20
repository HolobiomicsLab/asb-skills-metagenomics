---
name: fastqc-report-interpretation
description: Use when running or validating the short-read pre-processing module of
  nf-core/mag and you must judge, from FastQC reports, whether raw Illumina paired-end
  FASTQ files were adequately trimmed and contaminant-filtered before assembly.
when_to_use_negative:
- Input reads are long PacBio/Nanopore reads — nf-core/mag handles those with long-read
  tools (e.g. Filtlong, minimap2 alignment), not the fastp/FastQC short-read module
- Data are already quality-assessed feature outputs (e.g. assembled contigs or MAG
  bins) rather than raw/trimmed FASTQ
- The goal is de novo quality correction of reads — FastQC only profiles and reports;
  filtering is performed by fastp (and Bowtie2 contaminant removal)
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3173
tools:
- name: FastQC
  role: Generates quality-control reports on both the raw and the trimmed reads to
    document pre- and post-cleaning read quality
- name: fastp
  role: Performs adapter and quality trimming of short Illumina reads; its JSON/HTML
    QC report complements FastQC for interpreting before/after filtering metrics
  repo: https://github.com/OpenGene/fastp
- name: Bowtie2
  role: Short-read aligner used to identify and discard host or PhiX contaminant reads
    between the raw and final QC reports
- name: nf-test
  role: Executes the pre-processing stage with the test profile to verify the run
    completes and exits successfully
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:45562fd1bd4d15d9236e88fcc71f063cc1044caf71d55035bc4757a731b5b599
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - Bowtie2
  - FastQC
  - fastp
  - nf-core/mag
  - nf-test
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# fastqc-report-interpretation

## Summary

Interprets FastQC quality-control reports generated on both raw and pre-processed (fastp-trimmed, Bowtie2 host/PhiX-filtered) Illumina paired-end reads within the nf-core/mag metagenomics workflow, to verify each cleaning stage improved read quality before assembly and binning.

## When to use

Use when running or validating the short-read pre-processing module of nf-core/mag and you must judge, from FastQC reports, whether raw Illumina paired-end FASTQ files were adequately trimmed and contaminant-filtered before assembly.

## When NOT to use

- Input reads are long PacBio/Nanopore reads — nf-core/mag handles those with long-read tools (e.g. Filtlong, minimap2 alignment), not the fastp/FastQC short-read module
- Data are already quality-assessed feature outputs (e.g. assembled contigs or MAG bins) rather than raw/trimmed FASTQ
- The goal is de novo quality correction of reads — FastQC only profiles and reports; filtering is performed by fastp (and Bowtie2 contaminant removal)

## Inputs

- Raw Illumina paired-end FASTQ files (short reads, e.g. CAMISIM simulated metagenomic test data)
- Trimmed FASTQ files produced by the fastp step of nf-core/mag
- FastQC HTML/ZIP reports for raw and pre-processed reads

## Outputs

- Interpreted FastQC quality profiles (per-base quality, base content, GC ratio, duplication, adapter content) for pre- and post-cleaning reads
- Documented evidence that the pre-processing stage passed before downstream assembly and binning

## How to apply

Obtain the CAMISIM simulated metagenomic paired-end FASTQ inputs (Zenodo 10.5281/zenodo.5155395) and run the short-read branch of nf-core/mag (Nextflow DSL2) with the test profile so fastp applies adapter and quality trimming. Generate FastQC reports on both the raw and the trimmed reads, then compare the per-base quality curves, Q20/Q30, base contents, GC ratio, duplication and adapter-content panels between the two report sets to confirm trimming removed adapter contamination and low-quality tails. Check that reads surviving Bowtie2 alignment against PhiX and host references were discarded, so the post-cleaning reports reflect contaminant-depleted data. Confirm the stage end-to-end with the pipeline's test framework (nf-test) exiting successfully.

## Related tools

- **FastQC** (Generates quality-control reports on both the raw and the trimmed reads to document pre- and post-cleaning read quality)
- **fastp** (Performs adapter and quality trimming of short Illumina reads; its JSON/HTML QC report complements FastQC for interpreting before/after filtering metrics) — https://github.com/OpenGene/fastp
- **Bowtie2** (Short-read aligner used to identify and discard host or PhiX contaminant reads between the raw and final QC reports)
- **nf-test** (Executes the pre-processing stage with the test profile to verify the run completes and exits successfully) — https://github.com/nf-core/mag

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- FastQC reports exist for both raw and trimmed reads, allowing a direct pre-/post-cleaning comparison
- Adapter content and per-base quality panels improve after fastp trimming (adapters removed, low-quality 5'/3' tails cut)
- Post-Bowtie2 report reflects host/PhiX reads having been discarded
- The pre-processing stage completes and exits successfully under nf-test with the test and docker profiles

## Limitations

- FastQC only profiles and reports quality; it does not filter reads — trimming is fastp's role
- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read platforms (Nanopore, PacBio) require different tools (e.g. fastplong), so FastQC-report interpretation here applies only to the short-read branch
- Interpretation quality depends on representative test data; the pipeline is validated with CAMISIM simulated metagenomic data from Zenodo

## Evidence

- [intro] Typical metagenomic workflow includes read QC on raw and preprocessed reads: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [methods] Bowtie2 vs minimap2 for short vs long-read alignment: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [readme] fastp is an all-in-one FastQ preprocessing and QC tool: [evidence span withheld — anchor status: external_doc]
- [readme] fastp quality profiling covers metrics interpreted in QC reports: [evidence span withheld — anchor status: external_doc]
