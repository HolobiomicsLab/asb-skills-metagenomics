---
name: krona-community-composition-visualization
description: Use when preprocessed short-read FASTQ files must be screened for contamination
  and characterized taxonomically prior to metagenome assembly, using read-level classification
  with Krona charts and MultiQC aggregation.
when_to_use_negative:
- 'Input reads are long reads (Nanopore/PacBio): this QA stage is applied to short
  reads only, since not every tool is valid for every read type in nf-core/mag'
- The goal is taxonomic classification of assembled contigs or MAGs rather than of
  preprocessed reads
- A single-sample interactive visualization already exists and only cross-sample aggregation
  is needed — MultiQC aggregation alone would suffice without re-running Krona
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: nf-core/mag
  role: Pipeline hosting the read-classification QA step; gates tools by read type
    and defines ch_versions/ch_multiqc channel conventions
  repo: https://github.com/nf-core/mag
- name: Kraken2
  role: Primary tool classifying preprocessed short reads taxonomically; classification
    reports feed Krona rendering
- name: Centrifuge
  role: Alternative read classifier to Kraken2 for the same short-read QA step
- name: Krona
  role: Renders per-sample classification reports as interactive HTML charts of community
    composition
- name: MultiQC
  role: Aggregates per-sample classification and QC outputs into a single report configured
    via assets/multiqc_config.yml
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:3bf65fc54b2e38d104f30f3079788c4eaa345e2ac461a95bb588f972439cfb0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Centrifuge
  - Kraken2
  - Krona
  - MultiQC
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Krona community composition visualization

## Summary

Quality-assurance step in nf-core/mag in which preprocessed short reads are taxonomically classified with Kraken2 (or Centrifuge), per-sample classification reports are rendered as interactive Krona HTML charts, and results are aggregated into a single MultiQC report to assess contamination and microbial community composition before assembly.

## When to use

Use when preprocessed short-read FASTQ files must be screened for contamination and characterized taxonomically prior to metagenome assembly, using read-level classification with Krona charts and MultiQC aggregation.

## When NOT to use

- Input reads are long reads (Nanopore/PacBio): this QA stage is applied to short reads only, since not every tool is valid for every read type in nf-core/mag
- The goal is taxonomic classification of assembled contigs or MAGs rather than of preprocessed reads
- A single-sample interactive visualization already exists and only cross-sample aggregation is needed — MultiQC aggregation alone would suffice without re-running Krona

## Inputs

- Preprocessed short-read FASTQ files per sample
- Kraken2 or Centrifuge taxonomic database (assumed provisioned for read classification)
- assets/multiqc_config.yml MultiQC configuration

## Outputs

- Per-sample Kraken2/Centrifuge classification reports
- Interactive Krona HTML charts of community composition
- ch_versions channel with mixed tool version outputs
- multiqc_report.html aggregating classification and QC modules across all samples

## How to apply

Load the preprocessed short-read FASTQ files for each sample, noting that nf-core/mag gates tools by read type, so this classification step applies to short reads only and long-read-specific steps are kept out of this stage. Run Kraken2 (or alternatively Centrifuge) on the short reads to produce per-sample taxonomic classification reports, then convert these reports into interactive Krona HTML charts depicting community composition. Mix each tool's version output into ch_versions and route the classification/QC files into ch_multiqc following nf-core/mag channel conventions. Finally, render a single aggregate MultiQC report across all samples, ordering module plots, suffixes, and file-name cleanup according to assets/multiqc_config.yml.

## Related tools

- **nf-core/mag** (Pipeline hosting the read-classification QA step; gates tools by read type and defines ch_versions/ch_multiqc channel conventions) — https://github.com/nf-core/mag
- **Kraken2** (Primary tool classifying preprocessed short reads taxonomically; classification reports feed Krona rendering)
- **Centrifuge** (Alternative read classifier to Kraken2 for the same short-read QA step)
- **Krona** (Renders per-sample classification reports as interactive HTML charts of community composition)
- **MultiQC** (Aggregates per-sample classification and QC outputs into a single report configured via assets/multiqc_config.yml)

## Evaluation signals

- multiqc_report.html exists and is valid HTML
- multiqc_report.html contains a section for every processed sample
- The classification and QC modules appear within the report in the order configured in assets/multiqc_config.yml (suffixes, file-name cleanup, module plots)
- A Krona HTML chart is produced for each sample classified by Kraken2/Centrifuge
- All tool version outputs are present in ch_versions for the classification stage

## Limitations

- Only valid for short reads — long-read and hybrid steps must not enter this stage because not every tool is valid for every read type
- Classification results are used to assess potential contamination and community composition before assembly; they do not replace downstream assembly/binning QC
- Report content and plot ordering depend on keeping assets/multiqc_config.yml updated (suffixes, file-name cleanup, module order)

## Evidence

- [abstract] Pipeline scope includes taxonomic classification: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [intro] Position of QA step in the workflow: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [full_text] Read-classification QA with Krona visualization: [evidence span withheld — anchor status: not_found]
- [full_text] Channel conventions and MultiQC configuration: "Mix the version output channel into ch_versions and relevant files into ch_multiqc"
- [full_text] MultiQC config requirements: "Update MultiQC config assets/multiqc_config.yml so relevant suffixes, file name cleanup, and module plots are in the appropriate order"
