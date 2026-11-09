---
name: multiqc-aggregation-of-qc-outputs
description: Use when a pipeline stage produces many scattered per-sample QC/classification
  outputs (Kraken2/Centrifuge reports, Krona charts, preprocessing QC) and a single
  aggregated HTML report is needed to assess contamination before assembly.
when_to_use_negative:
- The input reads are long reads or hybrid data at this stage — classification/QC
  of this kind is applied to preprocessed short reads only, since not every tool is
  valid for every read type
- You need quantitative taxonomic abundance estimation rather than a QC-style contamination
  screen before assembly
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Kraken2
  role: Classifies preprocessed short reads by taxonomic origin to assess potential
    contamination before assembly
- name: Centrifuge
  role: Alternative read classifier to Kraken2 for short-read taxonomic classification
    in the QA step
- name: Krona
  role: Renders per-sample classification reports as interactive HTML charts of community
    composition
- name: MultiQC
  role: Aggregates the per-sample classification and QC outputs into a single HTML
    report ordered according to assets/multiqc_config.yml
- name: nf-core/mag
  role: Host pipeline implementing the QA step and the ch_versions / ch_multiqc channel
    conventions
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine (DSL2) in which the modules and channels are composed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:53c909ed8c58f7f349e3391a7c3f51e5b8f532903282e079d48443aede432a88
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multiqc-aggregation-of-qc-outputs

## Summary

Aggregates per-sample, per-tool QC outputs (e.g., Kraken2/Centrifuge read-classification reports, Krona charts, read-preprocessing QC) into a single interactive MultiQC HTML report within the nf-core/mag pipeline, enabling contamination assessment and QC review before metagenome assembly.

## When to use

Use when a pipeline stage produces many scattered per-sample QC/classification outputs (Kraken2/Centrifuge reports, Krona charts, preprocessing QC) and a single aggregated HTML report is needed to assess contamination before assembly.

## When NOT to use

- The input reads are long reads or hybrid data at this stage — classification/QC of this kind is applied to preprocessed short reads only, since not every tool is valid for every read type
- You need quantitative taxonomic abundance estimation rather than a QC-style contamination screen before assembly

## Inputs

- Preprocessed short-read FASTQ files (gzipped, per sample)
- Per-sample Kraken2 or Centrifuge classification reports
- Per-sample QC outputs from upstream preprocessing tools (e.g., fastp HTML/JSON QC reports)
- Tool version outputs (ch_versions channel)
- assets/multiqc_config.yml

## Outputs

- Interactive Krona HTML charts of per-sample community composition
- A single aggregate multiqc_report.html containing a section for every processed sample with the classification and QC modules in the configured order
- Aggregated version channels mixed into ch_versions

## How to apply

Run read classification on the preprocessed short-read FASTQ files with Kraken2 (or alternatively Centrifuge), then convert the per-sample classification reports into interactive Krona HTML charts depicting community composition. Follow nf-core/mag channel conventions: mix each tool's version output channel into ch_versions and route the relevant classification/QC files into ch_multiqc. Update assets/multiqc_config.yml so relevant suffixes, file-name cleanup, and module plots are in the appropriate order, since MultiQC parses and groups outputs by these file-name patterns. Finally render a single aggregate multiqc_report.html across all samples, which is used to judge potential contamination and the microbial community prior to the assembly step. Respect the pipeline's read-type gating: nf-core/mag processes short-read, long-read, and hybrid data, and not every tool is valid for every read type — classification of this kind applies to short reads.

## Related tools

- **Kraken2** (Classifies preprocessed short reads by taxonomic origin to assess potential contamination before assembly)
- **Centrifuge** (Alternative read classifier to Kraken2 for short-read taxonomic classification in the QA step)
- **Krona** (Renders per-sample classification reports as interactive HTML charts of community composition)
- **MultiQC** (Aggregates the per-sample classification and QC outputs into a single HTML report ordered according to assets/multiqc_config.yml)
- **nf-core/mag** (Host pipeline implementing the QA step and the ch_versions / ch_multiqc channel conventions) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine (DSL2) in which the modules and channels are composed)

## Evaluation signals

- multiqc_report.html exists and is valid HTML
- multiqc_report.html contains a section for every processed sample
- The classification and QC modules appear inside the report in the order configured in assets/multiqc_config.yml
- Each tool's version output was mixed into ch_versions (version channels are non-empty)
- Per-sample Krona charts render and depict the community composition consistent with the Kraken2/Centrifuge classification reports

## Limitations

- Read-type gating: not every tool is valid for every read type, so this QA step must not be applied to long reads at this stage
- The aggregation is only as good as the MultiQC config: suffixes, file-name cleanup, and module plot ordering must be maintained in assets/multiqc_config.yml or outputs may be misparsed or misgrouped
- Functional annotation as well as assembly and bin refinement steps are not yet included in the described pipeline version, so downstream interpretation via this report is limited to contamination/community assessment before assembly

## Evidence

- [abstract] nf-core/mag abstract scope: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [readme] QC visualization in aggregated HTML reports: [evidence span withheld — anchor status: external_doc]
- [abstract] Containerized reproducibility of the pipeline hosting the QA step: "The pipeline is easy to install-all dependencies are provided within containers-portable and reproducible."
