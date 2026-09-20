---
name: metagenomic-read-type-gating
description: Use when a metagenomic run mixes short-read-only, long-read-only, and
  paired short+long-read samples, and each sample must be routed to the correct assembly
  path (MEGAHIT vs SPAdes vs hybridSPAdes) before QUAST-based quality comparison.
when_to_use_negative:
- The dataset contains only short reads — gating adds no value; just run MEGAHIT or
  SPAdes directly.
- The dataset contains only long reads for a sample — hybridSPAdes is not applicable;
  a long-read assembler path is the wrong route for hybrid gating.
- Reads are raw and unpreprocessed — gating operates on preprocessed reads; run adapter/quality
  trimming (e.g. fastp, Porechop, Filtlong) first.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: nf-core/mag
  role: Nextflow/DSL2 pipeline that implements the read-type gating and dispatches
    each sample to the appropriate assembly process
  repo: https://github.com/nf-core/mag
- name: MEGAHIT
  role: De novo assembler used for short-read-only samples in the gated assembly branch
  repo: https://github.com/voutcn/megahit
- name: SPAdes
  role: Alternative de novo assembler for short-read-only samples
- name: hybridSPAdes
  role: Assembler dispatched for samples with both short and long reads, using long
    reads jointly with short reads
- name: QUAST
  role: Summarizes quality features of each gated assembly and feeds the merged quast_summary.tsv
    comparison
- name: nf-test
  role: Validates that each gated assembly process executes and exits successfully
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:5025a7d96f4473f232095d4ed508b5d97f794e709de16a8327f021884620c06c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - MEGAHIT
  - QUAST
  - SPAdes
  - hybridSPAdes
  - nf-core/mag
  - nf-test
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metagenomic-read-type-gating

## Summary

Classifies each metagenomic sample by its read content (short-read only, long-read only, or hybrid short+long) inside a Nextflow pipeline, and uses that classification to dispatch the sample to the correct de novo assembler (MEGAHIT, SPAdes, or hybridSPAdes) in the nf-core/mag workflow.

## When to use

Use when a metagenomic run mixes short-read-only, long-read-only, and paired short+long-read samples, and each sample must be routed to the correct assembly path (MEGAHIT vs SPAdes vs hybridSPAdes) before QUAST-based quality comparison.

## When NOT to use

- The dataset contains only short reads — gating adds no value; just run MEGAHIT or SPAdes directly.
- The dataset contains only long reads for a sample — hybridSPAdes is not applicable; a long-read assembler path is the wrong route for hybrid gating.
- Reads are raw and unpreprocessed — gating operates on preprocessed reads; run adapter/quality trimming (e.g. fastp, Porechop, Filtlong) first.

## Inputs

- Preprocessed short-read FASTQ files per sample
- Preprocessed long-read FASTQ files per sample (Oxford Nanopore or PacBio)
- Nextflow samplesheet/read-type configuration for the nf-core/mag run
- Assembler selection parameters (e.g. MEGAHIT vs SPAdes)

## Outputs

- Contig FASTA assemblies per sample per assembler (MEGAHIT, SPAdes, hybridSPAdes)
- Per-assembly QUAST quality summaries merged into quast_summary.tsv
- Passing nf-test execution of the assembly processes

## How to apply

Load the preprocessed short-read and long-read FASTQ files per sample and classify each sample by read type using the pipeline's read-type gating configuration in Nextflow (nf-core/mag, DSL2). Route short-read-only samples to the MEGAHIT de novo assembly process, with SPAdes as the alternative assembler selected via pipeline parameters. For samples carrying long reads, dispatch to hybridSPAdes so long reads are used jointly with short reads, running it in addition to the short-read-only assembly mode where applicable. After assembly, collect the per-assembler contig FASTA files and run QUAST on each, merging summaries into a single quast_summary.tsv so gated and hybrid assemblies can be compared. Validate the gating by executing the assembly processes with nf-test so each branch exits successfully.

## Related tools

- **nf-core/mag** (Nextflow/DSL2 pipeline that implements the read-type gating and dispatches each sample to the appropriate assembly process) — https://github.com/nf-core/mag
- **MEGAHIT** (De novo assembler used for short-read-only samples in the gated assembly branch) — https://github.com/voutcn/megahit
- **SPAdes** (Alternative de novo assembler for short-read-only samples)
- **hybridSPAdes** (Assembler dispatched for samples with both short and long reads, using long reads jointly with short reads)
- **QUAST** (Summarizes quality features of each gated assembly and feeds the merged quast_summary.tsv comparison)
- **nf-test** (Validates that each gated assembly process executes and exits successfully)

## Examples

```
nf-test test --tag test --profile +docker
```

## Evaluation signals

- Short-read-only samples produce MEGAHIT/SPAdes contig FASTA outputs and no hybridSPAdes output; samples with long reads additionally produce hybridSPAdes assemblies.
- QUAST summaries exist for every emitted assembly and merge without missing samples into a single quast_summary.tsv.
- Pipeline run completes with all gated assembly processes exiting successfully.
- nf-test run (e.g. tagged test, docker profile) passes for the assembly stage.
- Hybrid assemblies show improved continuity (longer contigs/N50) relative to short-read-only assemblies in the QUAST comparison, consistent with the hybrid rationale.

## Limitations

- Gating only classifies by presence of short vs long reads; read-type-specific filter steps still apply per platform (e.g. Porechop adapter trimming is skipped for PacBio reads).
- The article notes nf-core/mag at the presented version did not yet include assembly and bin refinement steps, which bound what follows the gated assembly output.
- Hybrid assembly increases cost/complexity; it is optional and only beneficial when long reads are available to increase assembly continuity.

## Evidence

- [abstract] Hybrid assembly dispatch for long-read samples: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [methods] Platform-specific filter gating within the pipeline: "Porechop adapter trimming is skipped for PacBio reads"
- [methods] Read-type-dependent alignment choice: "short- vs long-read alignment uses Bowtie2 vs minimap2"
- [readme] Long-read preprocessing tools in the gated workflow: [evidence span withheld — anchor status: external_doc]
- [intro] Hybrid assembly value proposition motivating gating: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost"
