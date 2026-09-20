---
name: short-read-quality-trimming
description: Use when raw paired-end Illumina FASTQ (e.g. CAMISIM simulated metagenomic
  test data) enters nf-core/mag and must be adapter/quality trimmed and decontaminated
  before assembly.
when_to_use_negative:
- 'Input is long-read data (Nanopore/PacBio): use the long-read branch (Porechop adapter
  trimming, Filtlong filtering, minimap2 alignment) instead of fastp/Bowtie2'
- Reads are already adapter- and quality-trimmed and contaminant-free; re-running
  trimming offers no benefit and can over-trim
- The goal is assembly or binning itself rather than read preprocessing — this skill
  only covers the trimming/decontamination stage
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: fastp
  role: Performs ultrafast all-in-one adapter and quality trimming (with automatic
    adapter detection) on the short Illumina reads
  repo: https://github.com/OpenGene/fastp
- name: Bowtie2
  role: Short-read aligner used to map trimmed reads against PhiX and host references
    so contaminant reads can be identified and removed
- name: FastQC
  role: Generates per-sample quality reports on the raw and trimmed reads to document
    pre- and post-cleaning quality
- name: nf-core/mag
  role: Best-practice Nextflow DSL2 pipeline that orchestrates the short-read preprocessing
    module
  repo: https://github.com/nf-core/mag
- name: nf-test
  role: Pipeline test framework used to verify the preprocessing stage runs and exits
    successfully
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:cf0528dd130238972955dfce65f9b8a9da25ad965b76b79eecd21930832a2fc0
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# short-read-quality-trimming

## Summary

Pre-process short Illumina reads within nf-core/mag by running fastp for adapter and quality trimming, then mapping with Bowtie2 to remove PhiX and host contaminants, with FastQC documenting quality before and after cleaning. This read-cleaning stage is the entry point of the reference-independent metagenomic workflow (preprocessing raw reads, assembly and binning).

## When to use

Use when raw paired-end Illumina FASTQ (e.g. CAMISIM simulated metagenomic test data) enters nf-core/mag and must be adapter/quality trimmed and decontaminated before assembly.

## When NOT to use

- Input is long-read data (Nanopore/PacBio): use the long-read branch (Porechop adapter trimming, Filtlong filtering, minimap2 alignment) instead of fastp/Bowtie2
- Reads are already adapter- and quality-trimmed and contaminant-free; re-running trimming offers no benefit and can over-trim
- The goal is assembly or binning itself rather than read preprocessing — this skill only covers the trimming/decontamination stage

## Inputs

- Raw paired-end Illumina FASTQ files (e.g. CAMISIM simulated metagenomic data, Zenodo 10.5281/zenodo.5155395)
- PhiX reference genome for contaminant mapping with Bowtie2
- Host reference genome for contaminant mapping with Bowtie2

## Outputs

- Adapter- and quality-trimmed FASTQ files (clean reads for downstream assembly)
- fastp QC report (JSON/HTML) summarizing filtering and trimming
- FastQC reports on both raw and trimmed reads
- Bowtie2 mapping results used to identify and discard host/PhiX reads

## How to apply

Place the Illumina paired-end FASTQ files in the pipeline input directory and run the short-read branch of nf-core/mag (Nextflow DSL2) with the test profile so fastp applies adapter and quality trimming to the raw reads. fastp detects adapter sequences automatically and can additionally cut low-quality bases per read by sliding-window evaluation and trim polyG/polyX 3' tails common in NovaSeq/NextSeq data. Map the trimmed reads with Bowtie2 — the short-read aligner of the pipeline, in contrast to minimap2 for long reads — against PhiX and host references and discard the mapped contaminant reads. Generate FastQC reports on both the raw and the trimmed reads to document pre- and post-cleaning quality. Verify the stage end-to-end by executing it with the pipeline's test framework (nf-test test --tag test --profile +docker) and confirming the run exits successfully.

## Related tools

- **fastp** (Performs ultrafast all-in-one adapter and quality trimming (with automatic adapter detection) on the short Illumina reads) — https://github.com/OpenGene/fastp
- **Bowtie2** (Short-read aligner used to map trimmed reads against PhiX and host references so contaminant reads can be identified and removed)
- **FastQC** (Generates per-sample quality reports on the raw and trimmed reads to document pre- and post-cleaning quality)
- **nf-core/mag** (Best-practice Nextflow DSL2 pipeline that orchestrates the short-read preprocessing module) — https://github.com/nf-core/mag
- **nf-test** (Pipeline test framework used to verify the preprocessing stage runs and exits successfully)

## Examples

```
fastp -i in_R1.fq.gz -I in_R2.fq.gz -o trimmed_R1.fq.gz -O trimmed_R2.fq.gz --detect_adapter_for_pe -- fastp. --html fastp.html
```

## Evaluation signals

- FastQC reports exist for both raw and trimmed reads and post-trim metrics show adapter content and low-quality tails reduced
- fastp report shows adapter sequences were trimmed and reads passing quality/length filters are emitted
- Reads mapping to PhiX or host references under Bowtie2 are absent from the cleaned FASTQ output
- nf-test test --tag test --profile +docker completes and the run exits successfully
- Cleaned FASTQ files are correctly passed to the downstream assembly step of the short-read branch

## Limitations

- fastp is designed for short reads (Illumina NovaSeq, MGI); the fastp README directs long-read users to fastplong instead
- This skill does not cover Porechop/Filtlong long-read trimming, which the pipeline applies to the long-read branch
- Removal of functional annotation, bin refinement, and metatranscriptomic steps is out of scope for the pipeline version described

## Evidence

- [methods] fastp trimming, Bowtie2 host/PhiX removal, FastQC pre/post QC: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [intro] Trimming is the first stage of the reference-independent metagenomic workflow: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning"
- [readme] fastp is the trimming tool for short reads: [evidence span withheld — anchor status: external_doc]
- [readme] Adapter detection is automatic in fastp: [evidence span withheld — anchor status: external_doc]
- [abstract] Pipeline dependencies are containerized for reproducibility of the trimming stage: "The pipeline is easy to install-all dependencies are provided within containers-portable and reproducible."
