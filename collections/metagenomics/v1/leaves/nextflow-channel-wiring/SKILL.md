---
name: nextflow-channel-wiring
description: Use when implementing or extending an nf-core/DSL2 pipeline module and
  you must connect a tool's outputs into pipeline-level version tracking and MultiQC
  aggregation channels.
when_to_use_negative:
- The tool applies to long reads or hybrid data at a stage where the pipeline gates
  it to short reads only (e.g. Kraken2/Centrifuge classification before assembly)
- The pipeline is not written in nf-core DSL2 with established ch_versions/ch_multiqc
  conventions — plain Snakemake or Nextflow scripts without channel conventions won't
  benefit
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: nf-core/mag
  role: Reference pipeline whose DSL2 channel conventions (ch_versions, ch_multiqc)
    define this wiring pattern
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine providing the DSL2 channel/mix semantics the skill wires modules
    with
- name: MultiQC
  role: Downstream consumer of the ch_multiqc channel, rendering the aggregate HTML
    report
- name: nf-core/modules
  role: Shared DSL2 module repository whose modules follow the versions/multiqc emission
    convention
  repo: https://github.com/nf-core/modules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:430002aacfa14b3a08167f0c6bf31f23e434c90e5fef8f918ce234e30547c895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Centrifuge
  - Kraken2
  - MultiQC
  - Nextflow
  - nf-core/mag
  - nf-core/modules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# nextflow-channel-wiring

## Summary

Wiring pattern for nf-core DSL2 modules, following the nf-core/mag convention of mixing each tool's version output into a shared ch_versions channel and routing per-tool result files (e.g. Kraken2/Centrifuge reports and Krona charts) into ch_multiqc so a single aggregate report is rendered.

## When to use

Use when implementing or extending an nf-core/DSL2 pipeline module and you must connect a tool's outputs into pipeline-level version tracking and MultiQC aggregation channels.

## When NOT to use

- The tool applies to long reads or hybrid data at a stage where the pipeline gates it to short reads only (e.g. Kraken2/Centrifuge classification before assembly)
- The pipeline is not written in nf-core DSL2 with established ch_versions/ch_multiqc conventions — plain Snakemake or Nextflow scripts without channel conventions won't benefit

## Inputs

- Preprocessed short-read FASTQ files per sample (classification stage input)
- nf-core DSL2 module output channels (versions channel, report/chart channels from Kraken2/Centrifuge/Krona)
- assets/multiqc_config.yml specifying suffixes, file-name cleanup, and module plot order

## Outputs

- ch_versions channel aggregating all tool version outputs
- ch_multiqc channel carrying classification and QC files
- single multiqc_report.html containing a section for every processed sample with modules in configured order
- per-sample Krona interactive HTML charts of community composition

## How to apply

Structure the module in nf-core's modular DSL2 style, as nf-core/mag does, so each sub-workflow emits its outputs on dedicated channels. After running a tool such as Kraken2 or Centrifuge on preprocessed short reads, mix the tool's version output channel into the pipeline-wide ch_versions channel, and mix the relevant result files (classification reports, Krona HTML charts, QC logs) into ch_multiqc. Gate tools by read type before wiring, since nf-core/mag processes short-read, long-read, and hybrid data and not every tool is valid for every read type — e.g. Kraken2/Centrifuge classification runs on short reads only. Ensure downstream aggregation (MultiQC) orders module plots, suffixes, and file-name cleanup via assets/multiqc_config.yml so the single HTML report reflects every processed sample in the configured order. The rationale is that the modular DSL2 structure nf-core/mag uses efficiently allows future extensions with new tools, and consistent channel conventions keep QC results aggregatable across samples.

## Related tools

- **nf-core/mag** (Reference pipeline whose DSL2 channel conventions (ch_versions, ch_multiqc) define this wiring pattern) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine providing the DSL2 channel/mix semantics the skill wires modules with)
- **MultiQC** (Downstream consumer of the ch_multiqc channel, rendering the aggregate HTML report)
- **nf-core/modules** (Shared DSL2 module repository whose modules follow the versions/multiqc emission convention) — https://github.com/nf-core/modules

## Examples

```
nextflow run nf-core/mag -profile test,docker
```

## Evaluation signals

- multiqc_report.html exists and is valid HTML after the run
- The MultiQC report contains a section for every processed sample
- All classification and QC modules (Kraken2/Centrifuge, Krona-derived plots) appear in the multiqc_config.yml-configured order
- Version outputs from every executed module are collected in ch_versions with no empty-mix errors (Nextflow does not fail silently on unemitted versions channels)
- No long-read-only channels are wired into short-read-only stages such as Kraken2/Centrifuge classification

## Limitations

- Not every tool is valid for every read type; channel wiring must respect read-type gating (short vs long vs hybrid) or invalid steps will run
- MultiQC plot ordering, suffix handling, and file-name cleanup depend on keeping assets/multiqc_config.yml in sync when adding new modules
- The modular DSL2 structure is specific to nf-core conventions; the pattern does not transfer to non-nf-core pipeline styles

## Evidence

- [discussion] The modular DSL2 structure efficiently allows future extensions with new tools: "The modular DSL2 structure efficiently allows future extensions with new tools"
- [intro] such as Snakemake (9) or Nextflow (10), which facilitate scalability, portability, reproducibility and ease of application: "such as Snakemake (9) or Nextflow (10), which facilitate scalability, portability, reproducibility and ease of application"
- [abstract] It is written in Nextflow and developed as part of the nf-core initiative for best-practice pipeline development.: "It is written in Nextflow and developed as part of the nf-core initiative for best-practice pipeline development."
