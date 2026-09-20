---
name: nextflow-pipeline-execution
description: Use when metagenomic FASTQ short and/or long reads must be processed
  end-to-end into assemblies and MAGs, or when benchmarking nf-core/mag assembly settings
  (hybrid, co-assembly) on simulated data such as CAMISIM mouse-gut time series.
when_to_use_negative:
- Input is already assembled (contig FASTA) or already binned MAGs — the pipeline's
  preprocessing/assembly stages would not apply
- Reference-guided strain analysis requiring taxonomic profiling only, where a lighter
  classification workflow suffices
- Analyses requiring functional annotation or bin refinement — explicitly not yet
  included in nf-core/mag v2.1.0
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0769
tools:
- name: nf-core/mag
  role: The metagenome hybrid assembly and binning pipeline being executed; modular
    DSL2 structure dispatches preprocessing, assembly and binning subworkflows
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow management system that executes the pipeline with scalability, portability
    and reproducibility
  repo: https://nextflow.io/
- name: nf-core/tools
  role: Provides nf-core pipeline infrastructure and template conventions used by
    nf-core/mag
  repo: https://github.com/nf-core/tools
- name: nf-test
  role: Test framework used to validate pipeline/stage execution exits successfully
    on test data
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:93d7e78c33da6cf5855bafa6de4a36a176ccd176bd34e235592b037e83e73ace
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Filtlong
  - MEGAHIT
  - Nextflow
  - Porechop
  - QUAST
  - SPAdes
  - hybridSPAdes
  - minimap2
  - nf-core/mag
  - nf-core/tools
  - nf-test
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nextflow-pipeline-execution

## Summary

Executes the nf-core/mag metagenome assembly and binning pipeline as reproducible Nextflow DSL2 workflows, covering read preprocessing (fastp, Porechop, Filtlong), assembly (MEGAHIT, SPAdes, hybridSPAdes), binning and taxonomic classification, with dependencies supplied via containers.

## When to use

Use when metagenomic FASTQ short and/or long reads must be processed end-to-end into assemblies and MAGs, or when benchmarking nf-core/mag assembly settings (hybrid, co-assembly) on simulated data such as CAMISIM mouse-gut time series.

## When NOT to use

- Input is already assembled (contig FASTA) or already binned MAGs — the pipeline's preprocessing/assembly stages would not apply
- Reference-guided strain analysis requiring taxonomic profiling only, where a lighter classification workflow suffices
- Analyses requiring functional annotation or bin refinement — explicitly not yet included in nf-core/mag v2.1.0

## Inputs

- Raw paired-end Illumina FASTQ files (e.g. CAMISIM simulated mouse-gut time-series short reads, Zenodo 10.5281/zenodo.5155395)
- Raw Nanopore or PacBio long-read FASTQ files (hybrid runs)
- Host and PhiX reference sequences for contaminant read removal
- Pipeline configuration: samplesheet, Nextflow profile (docker/conda) and parameters (e.g. assembler choice, co-assembly grouping)

## Outputs

- Quality- and adapter-trimmed, decontaminated FASTQ files per read type
- De novo assemblies (MEGAHIT/SPAdes/hybridSPAdes contig FASTAs) with QUAST quality summaries (quast_summary.tsv)
- Reconstructed MAGs from the binning step with QC reports
- FastQC/NanoPlot reports for raw and preprocessed reads

## How to apply

Configure the run with a samplesheet of raw Illumina paired-end and Nanopore/PacBio FASTQ inputs and launch via Nextflow with an appropriate profile (e.g. test,docker). Read-type gating dispatches preprocessing correctly: fastp trimming plus Bowtie2-based host/PhiX removal and FastQC for short reads, versus Porechop adapter trimming, NanoLyse lambda-control removal, Filtlong quality filtering and NanoPlot QC for Nanopore reads (Porechop is skipped for PacBio). Assembly dispatches to MEGAHIT or SPAdes for short-read-only input and to hybridSPAdes when long reads are present, aligning short reads with Bowtie2 and long reads with minimap2; group-wise co-assembly can pool samples to exploit sample-wise group information. The modular DSL2 structure supports extension, and all dependencies are provided within containers, making runs portable and reproducible. Validate the run with the pipeline's test framework (nf-test with docker profile) and confirm stage outputs such as quality-filtered FASTQs, QUAST summaries and binning output are produced.

## Related tools

- **nf-core/mag** (The metagenome hybrid assembly and binning pipeline being executed; modular DSL2 structure dispatches preprocessing, assembly and binning subworkflows) — https://github.com/nf-core/mag
- **Nextflow** (Workflow management system that executes the pipeline with scalability, portability and reproducibility) — https://nextflow.io/
- **nf-core/tools** (Provides nf-core pipeline infrastructure and template conventions used by nf-core/mag) — https://github.com/nf-core/tools
- **nf-test** (Test framework used to validate pipeline/stage execution exits successfully on test data)

## Examples

```
nf-test test --tag test --profile +docker
```

## Evaluation signals

- nf-test run (e.g. 'nf-test test --tag test --profile +docker') completes and exits successfully on the pipeline test data
- Quality-filtered FASTQ files are present as outputs of each preprocessing stage
- QUAST summaries merge into a single comparison table covering every assembly produced across the four assembly settings
- Benchmarked metrics (assembly size, N50, MAG count) reproduce the reported trend: hybrid and co-assembly increase size, N50 and MAG recovery
- Read-type gating is respected: Porechop skipped for PacBio reads, Bowtie2 used for short reads and minimap2 for long reads

## Limitations

- Functional annotation, assembly refinement and bin refinement steps were not yet included in the presented version 2.1.0
- Metatranscriptomic data analysis workflow not yet integrated
- Only Muffin among comparable pipelines supported hybrid assembly and none supported co-assembly at time of writing; results depend on correct read-type gating and co-assembly grouping configuration

## Evidence

- [abstract] Pipeline written in Nextflow with containerized dependencies: "It is written in Nextflow and developed as part of the nf-core initiative for best-practice pipeline development."
- [abstract] Reproducibility via containerized dependencies: "The pipeline is easy to install-all dependencies are provided within containers-portable and reproducible."
- [discussion] Modular DSL2 structure enabling extension: "The modular DSL2 structure efficiently allows future extensions with new tools"
- [methods] Read-type-specific aligners for preprocessing: "short- vs long-read alignment uses Bowtie2 vs minimap2"
