---
name: adapter-and-contaminant-read-filtering
description: Use when raw Illumina paired-end FASTQ metagenomic reads must be cleaned
  before assembly — adapters, low-quality bases, PhiX spike-in, or host contamination
  would degrade downstream MAG assembly and binning.
when_to_use_negative:
- Input is long-read (PacBio/Nanopore) data — the pipeline uses Porechop adapter trimming
  and minimap2 alignment for long reads instead of fastp and Bowtie2
- Reads are already adapter-trimmed and quality-filtered and only require assembly
  or binning
- PacBio reads are the input — Porechop adapter trimming is skipped for PacBio reads,
  so adapter-trimming logic does not apply
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: fastp
  role: Performs adapter and quality trimming of short Illumina reads (ultrafast all-in-one
    preprocessing for FASTQ data)
  repo: https://github.com/OpenGene/fastp
- name: Bowtie2
  role: Short-read aligner used to map trimmed reads against PhiX and host references
    so contaminant reads can be identified and removed
- name: FastQC
  role: Quality control reports on both raw and preprocessed reads to document pre-
    and post-cleaning quality
- name: nf-core/mag
  role: Pipeline implementing the short-read pre-processing module in Nextflow DSL2
  repo: https://github.com/nf-core/mag
- name: nf-test
  role: Test framework used to verify the pre-processing stage end-to-end (nf-test
    test --tag test --profile +docker)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:8121b4ce78e1bb478fa6e9acdf02f9fcd6f2341cd5d57ec96214cac90352a114
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# adapter-and-contaminant-read-filtering

## Summary

Pre-processes short Illumina metagenomic reads in the nf-core/mag pipeline by trimming adapters and low-quality bases with fastp, then mapping reads with Bowtie2 against PhiX and host references to discard contaminant reads, with FastQC documenting pre- and post-cleaning quality.

## When to use

Use when raw Illumina paired-end FASTQ metagenomic reads must be cleaned before assembly — adapters, low-quality bases, PhiX spike-in, or host contamination would degrade downstream MAG assembly and binning.

## When NOT to use

- Input is long-read (PacBio/Nanopore) data — the pipeline uses Porechop adapter trimming and minimap2 alignment for long reads instead of fastp and Bowtie2
- Reads are already adapter-trimmed and quality-filtered and only require assembly or binning
- PacBio reads are the input — Porechop adapter trimming is skipped for PacBio reads, so adapter-trimming logic does not apply

## Inputs

- Raw Illumina paired-end FASTQ short reads (e.g. CAMISIM simulated metagenomic data, Zenodo 10.5281/zenodo.5155395)
- Host reference genome(s) for contaminant removal
- PhiX control reference for spike-in removal

## Outputs

- Adapter- and quality-trimmed FASTQ reads
- Contaminant-filtered (host- and PhiX-depleted) FASTQ reads
- fastp QC/trimming reports (JSON/HTML)
- FastQC reports for raw and preprocessed reads
- Bowtie2 mapping output used to identify contaminant reads

## How to apply

Place the Illumina paired-end FASTQ files (e.g. the CAMISIM simulated metagenomic test data from Zenodo 10.5281/zenodo.5155395) in the pipeline input directory and run the short-read branch of nf-core/mag (Nextflow DSL2) with the test profile so fastp applies adapter and quality trimming to the raw reads. fastp performs automatic adapter detection, sliding-window quality cutting, and length/complexity filtering on the FASTQ data. The trimmed reads are then mapped with Bowtie2 — the short-read aligner of the pipeline, in contrast to minimap2 for long reads — against PhiX and host references, and mapped contaminant reads are discarded. FastQC reports are generated on both raw and trimmed reads to document pre- and post-cleaning quality. Verify the stage end-to-end by running it with the pipeline's test framework (nf-test test --tag test --profile +docker) and confirming the run completes and exits successfully.

## Related tools

- **fastp** (Performs adapter and quality trimming of short Illumina reads (ultrafast all-in-one preprocessing for FASTQ data)) — https://github.com/OpenGene/fastp
- **Bowtie2** (Short-read aligner used to map trimmed reads against PhiX and host references so contaminant reads can be identified and removed)
- **FastQC** (Quality control reports on both raw and preprocessed reads to document pre- and post-cleaning quality)
- **nf-core/mag** (Pipeline implementing the short-read pre-processing module in Nextflow DSL2) — https://github.com/nf-core/mag
- **nf-test** (Test framework used to verify the pre-processing stage end-to-end (nf-test test --tag test --profile +docker))

## Examples

```
nf-test test --tag test --profile docker
```

## Evaluation signals

- The Nextflow run of the short-read pre-processing module completes and exits successfully (nf-test test --tag test --profile +docker passes)
- FastQC reports are generated for both raw and trimmed reads and show improved quality after cleaning
- Reads mapping to PhiX and host references with Bowtie2 are absent from the final preprocessed FASTQ output
- fastp trimming report confirms adapter detection and quality/length filtering were applied to the input FASTQ files
- Trimmed, contaminant-free reads serve as valid input to the downstream assembly step of the metagenomic workflow

## Limitations

- The fastp-based trimming strategy is designed for short reads (Illumina NovaSeq, MGI); long reads require different tools (Porechop/Filtlong, fastplong)
- Contaminant removal only covers references supplied (host and PhiX); other contaminants pass through
- Adapter trimming with Porechop is skipped for PacBio reads, so the same filtering rules cannot be applied uniformly across sequencing platforms

## Evidence

- [methods] short- vs long-read alignment uses Bowtie2 vs minimap2: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [readme] fastp purpose: [evidence span withheld — anchor status: external_doc]
- [readme] fastp adapter detection: [evidence span withheld — anchor status: external_doc]
- [methods] Porechop skipped for PacBio: "Porechop adapter trimming is skipped for PacBio reads"
- [supplementary] CAMISIM test data availability: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
