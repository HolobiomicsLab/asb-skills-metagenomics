---
name: metagenomic-read-taxonomic-classification
description: 'Use when quality-assuring preprocessed short-read FASTQ metagenomic
  samples prior to assembly: to classify reads against a reference database, detect
  contamination, and visualize microbial community composition per sample with Krona
  charts and a MultiQC report.'
when_to_use_negative:
- The input reads are long reads (e.g., Nanopore/PacBio) at this stage — nf-core/mag
  explicitly gates tools by read type and this classification/QC step is scoped to
  preprocessed short reads only
- The goal is taxonomic classification of assembled contigs or MAG bins rather than
  individual preprocessed reads
- Samples have already been classified and the task is downstream genome binning or
  MAG annotation
edam_operation: http://edamontology.org/operation_0290
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: nf-core/mag
  role: Host pipeline that executes the read-classification quality-assurance stage
    and orchestrates channels (ch_versions, ch_multiqc)
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine in which the modular DSL2 classification modules are executed
    and version channels mixed
  repo: https://github.com/nf-core/mag
- name: Kraken2
  role: Primary k-mer-based tool classifying preprocessed short reads against a reference
    database
- name: Centrifuge
  role: Alternative classifier to Kraken2 for short-read taxonomic assignment
- name: Krona
  role: Renders per-sample classification reports as interactive HTML charts of community
    composition
- name: MultiQC
  role: Aggregates per-sample classification and QC outputs into a single HTML report
    configured via assets/multiqc_config.yml
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:22c0e0db69f25669a392db022697a210e9c6719d6b7e1de881e81e4dcc733114
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Centrifuge
  - Kraken2
  - Krona
  - MultiQC
  - Nextflow
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0290
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metagenomic-read-taxonomic-classification

## Summary

Classifies preprocessed metagenomic short reads taxonomically with Kraken2 or Centrifuge inside the nf-core/mag pipeline, rendering per-sample Krona charts and an aggregated MultiQC report to assess contamination and community composition before assembly.

## When to use

Use when quality-assuring preprocessed short-read FASTQ metagenomic samples prior to assembly: to classify reads against a reference database, detect contamination, and visualize microbial community composition per sample with Krona charts and a MultiQC report.

## When NOT to use

- The input reads are long reads (e.g., Nanopore/PacBio) at this stage — nf-core/mag explicitly gates tools by read type and this classification/QC step is scoped to preprocessed short reads only
- The goal is taxonomic classification of assembled contigs or MAG bins rather than individual preprocessed reads
- Samples have already been classified and the task is downstream genome binning or MAG annotation

## Inputs

- Preprocessed short-read FASTQ files (R1/R2) per sample
- Taxonomic reference database for Kraken2 or Centrifuge
- nf-core/mag pipeline module definitions (Kraken2/Centrifuge, Krona, MultiQC)
- assets/multiqc_config.yml with suffix, file-name cleanup, and module plot ordering

## Outputs

- Per-sample Kraken2/Centrifuge classification reports
- Interactive Krona HTML charts of per-sample community composition
- ch_versions channel with mixed tool version outputs
- ch_multiqc channel with classification and QC files
- multiqc_report.html aggregate report containing a section for every processed sample

## How to apply

Load the preprocessed short-read FASTQ files for each sample; note that nf-core/mag gates tools by read type, so this classification stage runs on short reads only, with long-read-specific steps excluded. Run Kraken2 (or, alternatively, Centrifuge) on the short reads to assign taxonomy to each read. Convert the per-sample classification reports into interactive Krona HTML charts that depict community composition. Following nf-core/mag channel conventions, mix each module's version output into ch_versions and route the classification/QC output files into ch_multiqc. Finally render a single aggregate multiqc_report.html across all samples, with suffixes, file-name cleanup, and module plots configured per assets/multiqc_config.yml. The rationale is that read-level taxonomic profiling before assembly exposes potential contamination and the expected microbial community, so downstream MAG recovery can be interpreted in context.

## Related tools

- **nf-core/mag** (Host pipeline that executes the read-classification quality-assurance stage and orchestrates channels (ch_versions, ch_multiqc)) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine in which the modular DSL2 classification modules are executed and version channels mixed) — https://github.com/nf-core/mag
- **Kraken2** (Primary k-mer-based tool classifying preprocessed short reads against a reference database)
- **Centrifuge** (Alternative classifier to Kraken2 for short-read taxonomic assignment)
- **Krona** (Renders per-sample classification reports as interactive HTML charts of community composition)
- **MultiQC** (Aggregates per-sample classification and QC outputs into a single HTML report configured via assets/multiqc_config.yml)

## Evaluation signals

- multiqc_report.html exists and is valid HTML
- multiqc_report.html contains a section for every processed sample
- The classification and QC modules appear in multiqc_report.html in the order configured in assets/multiqc_config.yml
- A Krona HTML chart is produced for each sample and reflects the per-sample Kraken2/Centrifuge classification report
- Tool version outputs are correctly mixed into ch_versions (nf-core/mag channel convention)

## Limitations

- Not every tool in nf-core/mag is valid for every read type: this classification/QC stage applies to short reads only, so long-read and hybrid inputs must route through different modules
- Classification quality depends on the supplied Kraken2/Centrifuge reference database; results are read-level profiles used for contamination screening, not MAG-level taxonomic annotation
- MultiQC report correctness depends on keeping assets/multiqc_config.yml suffixes, file-name cleanup, and module plots in the appropriate order

## Evidence

- [methods] nf-core/mag classifies preprocessed short reads with Kraken2 or Centrifuge, rendering the results as Krona charts: "nf-core/mag processes short-read, long-read, and hybrid data, and not every tool is valid for every read type"
- [abstract] The pipeline's overall purpose includes taxonomic classification of metagenomic reads: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [supplementary] Aggregation of module outputs follows MultiQC configuration conventions: "nf-core/mag code is hosted on GitHub under the nf-core organization https://github.com/nf-core/mag and released under the MIT license."
