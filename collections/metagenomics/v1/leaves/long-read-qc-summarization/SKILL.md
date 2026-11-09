---
name: long-read-qc-summarization
description: Use when raw Nanopore FASTQ reads enter the nf-core/mag long-read pre-processing
  subworkflow and per-sample QC reports are needed before and after adapter trimming,
  control decontamination and Filtlong quality filtering.
when_to_use_negative:
- The input is short Illumina reads — nf-core/mag handles short-read QC with different
  modules (e.g. FastQC/fastp-style preprocessing), not the long-read Porechop/Filtlong/NanoPlot
  chain
- The input is PacBio reads — Porechop adapter trimming is skipped for PacBio reads
  per nf-core/mag conventions, so the Nanopore-specific trimming/QC gating does not
  apply unchanged
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoPlot
  role: Generates QC summary statistics and length-vs-quality plots for raw and filtered
    long reads
- name: Porechop
  role: Trims Oxford Nanopore adapters from long reads prior to decontamination and
    QC summarization
  repo: https://github.com/rrwick/Porechop
- name: Filtlong
  role: Filters long reads by quality (length and identity) to produce the filtered
    FASTQ summarized in the QC reports
  repo: https://github.com/rrwick/Filtlong
- name: nf-core/mag
  role: Nextflow pipeline hosting the long-read pre-processing subworkflow in which
    QC summarization runs
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine executing the pre-processing and QC steps reproducibly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:7a61c4c59814c90b20ccbb4258582e9b53188c4acb7ee146fc257a39b5fd1e97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rrwick/Porechop
  tools:
  - Filtlong
  - NanoPlot
  - Nextflow
  - Porechop
  - nf-core/mag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# long-read-qc-summarization

## Summary

Generate QC summary statistics and plots for raw and quality-filtered long Nanopore reads within the nf-core/mag pre-processing module, using NanoPlot so that adapter trimming, lambda-control removal and Filtlong filtering can be validated before assembly.

## When to use

Use when raw Nanopore FASTQ reads enter the nf-core/mag long-read pre-processing subworkflow and per-sample QC reports are needed before and after adapter trimming, control decontamination and Filtlong quality filtering.

## When NOT to use

- The input is short Illumina reads — nf-core/mag handles short-read QC with different modules (e.g. FastQC/fastp-style preprocessing), not the long-read Porechop/Filtlong/NanoPlot chain
- The input is PacBio reads — Porechop adapter trimming is skipped for PacBio reads per nf-core/mag conventions, so the Nanopore-specific trimming/QC gating does not apply unchanged

## Inputs

- raw Nanopore long-read FASTQ files
- Filtlong quality-filtered long-read FASTQ files

## Outputs

- NanoPlot QC summary report (HTML with read length and quality plots) for the raw read set
- NanoPlot QC summary report for the filtered read set
- quality-filtered long-read FASTQ delivered as the pre-processing stage output

## How to apply

Within the nf-core/mag long-read pre-processing subworkflow (Nextflow DSL2), run NanoPlot on the raw Nanopore FASTQ input and again on the Filtlong quality-filtered FASTQ output, producing HTML summary reports of read length and Phred quality distributions. Sequence the steps so Porechop adapter trimming (skipped for PacBio reads) precedes NanoLyse removal of phage lambda control reads, and Filtlong filtering precedes the second NanoPlot pass, so both raw and filtered read sets are summarized. Use the paired before/after reports to confirm that trimming, decontamination and quality filtering changed the read set as expected. Validation requires that the GitHub Actions pipeline tests exit successfully on test data and that the quality-filtered FASTQ is present as the stage output alongside its QC report.

## Related tools

- **NanoPlot** (Generates QC summary statistics and length-vs-quality plots for raw and filtered long reads)
- **Porechop** (Trims Oxford Nanopore adapters from long reads prior to decontamination and QC summarization) — https://github.com/rrwick/Porechop
- **Filtlong** (Filters long reads by quality (length and identity) to produce the filtered FASTQ summarized in the QC reports) — https://github.com/rrwick/Filtlong
- **nf-core/mag** (Nextflow pipeline hosting the long-read pre-processing subworkflow in which QC summarization runs) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine executing the pre-processing and QC steps reproducibly)

## Examples

```
nextflow run nf-core/mag --input samplesheet.csv --platform 'ont' --outdir results -profile test,docker
```

## Evaluation signals

- GitHub Actions pipeline tests run the pipeline on test data and exit successfully
- The quality-filtered FASTQ is present as the stage output after pre-processing
- NanoPlot reports exist for both the raw and the filtered read sets
- Filtered-read NanoPlot report shows improved read quality relative to the raw-read report, consistent with Filtlong filtering
- No phage lambda control contamination remains detectable in the reads downstream of NanoLyse

## Limitations

- Porechop adapter trimming is gated to Nanopore reads and skipped for PacBio reads, so QC summaries around the trimming step are Nanopore-specific
- The paper's presented version 2.1.0 of nf-core/mag is under continuous development, so module versions and defaults may change over time

## Evidence

- [methods] For long Nanopore reads, nf-core/mag pre-processing uses porechop for adapter trimming and NanoLyse to remove phage lambda control contamination.: [evidence span withheld — anchor status: not_found]
- [methods] Porechop adapter trimming is skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [intro] A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [readme] NanoPlot is a plotting tool for long read sequencing data: [evidence span withheld — anchor status: external_doc]
- [readme] Filtlong uses both read length and read identity when choosing which reads pass the filter: [evidence span withheld — anchor status: external_doc]
