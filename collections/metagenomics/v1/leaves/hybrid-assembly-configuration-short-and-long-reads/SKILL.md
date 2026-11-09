---
name: hybrid-assembly-configuration-short-and-long-reads
description: Use when you have paired short-read FASTQ plus matching long-read FASTQ/FASTA
  from the same metagenomic samples (e.g. CAMISIM-simulated mouse-gut time-series
  data) and want to improve assembly N50, size, and MAG recovery.
when_to_use_negative:
- Only short reads are available and no long-read data exist for the samples — the
  hybrid configuration cannot be applied and short-read-only assembly should be used
- Reads are metatranscriptomic rather than metagenomic — the article notes this workflow
  does not yet cover metatranscriptomic data analysis
- The goal is bin refinement or functional annotation of existing assemblies — these
  steps are not yet included in nf-core/mag v2.1.0
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: nf-core/mag
  role: Best-practice Nextflow pipeline executing the hybrid assembly, binning and
    taxonomic classification workflow
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow management system used to configure and launch nf-core/mag runs across
    the four assembly settings
- name: Bowtie2
  role: Aligner applied to short reads during the hybrid assembly read-type-gated
    mapping step
- name: minimap2
  role: Aligner applied to long reads during the hybrid assembly read-type-gated mapping
    step
- name: Porechop
  role: Adapter trimming for long reads; skipped for PacBio reads per the pipeline's
    read-type gating
  repo: https://github.com/rrwick/Porechop
- name: Filtlong
  role: Quality filtering of long reads by length and read identity before assembly
  repo: https://github.com/rrwick/Filtlong
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:87811a8aea5a1219b347a3a2329f624d72e5e33ddd2003871d47fde904c66465
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - Bowtie2
  - Filtlong
  - Nextflow
  - Porechop
  - minimap2
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hybrid-assembly-configuration-short-and-long-reads

## Summary

Configure nf-core/mag v2.1.0 for hybrid metagenome assembly that combines Illumina short reads with long reads (Nanopore/PacBio) to increase assembly continuity, including the read-type-specific alignment and preprocessing gating (Bowtie2 for short reads, minimap2 for long reads).

## When to use

Use when you have paired short-read FASTQ plus matching long-read FASTQ/FASTA from the same metagenomic samples (e.g. CAMISIM-simulated mouse-gut time-series data) and want to improve assembly N50, size, and MAG recovery.

## When NOT to use

- Only short reads are available and no long-read data exist for the samples — the hybrid configuration cannot be applied and short-read-only assembly should be used
- Reads are metatranscriptomic rather than metagenomic — the article notes this workflow does not yet cover metatranscriptomic data analysis
- The goal is bin refinement or functional annotation of existing assemblies — these steps are not yet included in nf-core/mag v2.1.0

## Inputs

- Paired-end short-read FASTQ files (Illumina) per sample
- Long-read FASTQ/FASTA files (Oxford Nanopore or PacBio) per sample
- Samplesheet mapping samples to both read types and, for co-assembly, group assignments
- nf-core/mag v2.1.0 pipeline configuration (Nextflow params)

## Outputs

- Hybrid contig assemblies per sample or per co-assembly group
- Assembly statistics with size and N50 per setting
- Binned MAGs per setting from the binning output
- Comparison table (e.g. mag_assembly_benchmark_table.csv) across the four settings

## How to apply

Download the short- and long-read CAMISIM data (Zenodo DOI 10.5281/zenodo.5155395) and build a samplesheet assigning both read types per sample. Run nf-core/mag v2.1.0 via Nextflow with hybrid assembly enabled; the pipeline applies read-type gating automatically — Porechop adapter trimming is skipped for PacBio reads, short reads are aligned with Bowtie2 and long reads with minimap2. Decide between sample-wise assembly and group-wise co-assembly (pooling samples exploits sample-wise group information and higher sequencing depth); run both when benchmarking. Repeat the run in non-hybrid (short-read-only) mode to obtain the comparison baseline. Extract assembly size, N50, and MAG counts from the assembly statistics and binning outputs, and confirm that hybrid and co-assembly settings improve these metrics, re-checking read-type gating if any setting deviates.

## Related tools

- **nf-core/mag** (Best-practice Nextflow pipeline executing the hybrid assembly, binning and taxonomic classification workflow) — https://github.com/nf-core/mag
- **Nextflow** (Workflow management system used to configure and launch nf-core/mag runs across the four assembly settings)
- **Bowtie2** (Aligner applied to short reads during the hybrid assembly read-type-gated mapping step)
- **minimap2** (Aligner applied to long reads during the hybrid assembly read-type-gated mapping step)
- **Porechop** (Adapter trimming for long reads; skipped for PacBio reads per the pipeline's read-type gating) — https://github.com/rrwick/Porechop
- **Filtlong** (Quality filtering of long reads by length and read identity before assembly) — https://github.com/rrwick/Filtlong

## Examples

```
nextflow run nf-core/mag -r 2.1.0 --input samplesheet.csv --outdir results --assembly megahit
```

## Evaluation signals

- Hybrid assembly settings yield larger assembly size and higher N50 than the short-read-only setting on the CAMISIM time-series data
- Hybrid and co-assembly settings recover more MAGs in the binning output than short-read sample-wise assembly
- Read-type gating is correct: Porechop adapter trimming was skipped for PacBio reads, and Bowtie2 vs minimap2 were applied to short vs long reads respectively
- The four settings (short/hybrid × sample-wise/co-assembly) each produce a complete set of assembly statistics and MAG counts for the benchmark table
- Relative ordering of settings matches the trend reported in Figure 2; any deviating run is re-checked for read-type gating and co-assembly grouping before acceptance

## Limitations

- nf-core/mag v2.1.0 does not yet include functional annotation, assembly refinement, or bin refinement steps
- Only Muffin among comparable pipelines supports hybrid assembly and none supports co-assembly, limiting cross-pipeline comparison options
- Benchmark conclusions are grounded in CAMISIM-simulated mouse-gut time-series data and may not transfer directly to real datasets

## Evidence

- [abstract] Hybrid assembly combines short and long read advantages: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [intro] Hybrid assembly rationale: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost"
- [methods] Read-type-gated aligners: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [methods] Porechop gating for PacBio: "Porechop adapter trimming is skipped for PacBio reads"
- [supplementary] Benchmark dataset: "with CAMISIM simulated metagenomic data used to generate the exemplary results is available at https://doi.org/10.5281/zenodo.5155395"
