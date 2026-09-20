---
name: metagenome-de-novo-assembly
description: Use when preprocessed metagenomic FASTQ reads (short-only, long-only,
  or hybrid short+long) per sample must be assembled de novo into contigs and per-assembler
  quality summaries are needed.
when_to_use_negative:
- Input reads are still raw/unpreprocessed — the assembler expects preprocessed FASTQ
  files, not raw sequencing output
- A complete reference genome exists and a reference-guided (mapping-based) approach
  would be more appropriate than reference-independent de novo assembly
- The goal is binning or taxonomic classification of existing contigs rather than
  producing the assemblies themselves
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: nf-core/mag
  role: Nextflow (DSL2) pipeline that gates samples by read type and dispatches each
    sample to the correct assembly process
  repo: https://github.com/nf-core/mag
- name: MEGAHIT
  role: Primary de novo assembler for short-read-only metagenomic samples
  repo: https://github.com/voutcn/megahit
- name: SPAdes
  role: Alternative de novo assembler for short-read-only samples
- name: hybridSPAdes
  role: Hybrid assembler invoked when long reads are present, jointly using long and
    short reads
- name: QUAST
  role: Summarizes quality features of each assembly and feeds the merged quast_summary.tsv
    comparison table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:9c6278a2de1de8ac1479e1506d88f2ea6055f8e1ba9cfd1b94c3da7bc4727c9c
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metagenome-de-novo-assembly

## Summary

Reconstructs metagenome-assembled genome (MAG) contigs by de novo assembly of preprocessed short reads with MEGAHIT or SPAdes inside the nf-core/mag Nextflow pipeline, dispatching to hybridSPAdes when long reads are available, and assessing assembly quality with QUAST.

## When to use

Use when preprocessed metagenomic FASTQ reads (short-only, long-only, or hybrid short+long) per sample must be assembled de novo into contigs and per-assembler quality summaries are needed.

## When NOT to use

- Input reads are still raw/unpreprocessed — the assembler expects preprocessed FASTQ files, not raw sequencing output
- A complete reference genome exists and a reference-guided (mapping-based) approach would be more appropriate than reference-independent de novo assembly
- The goal is binning or taxonomic classification of existing contigs rather than producing the assemblies themselves

## Inputs

- Preprocessed short-read FASTQ files per sample
- Preprocessed long-read FASTQ files per sample (Oxford Nanopore or PacBio, for hybrid samples)
- Per-sample read-type classification (short-only / long-only / hybrid) from the pipeline's gating configuration

## Outputs

- Contig FASTA files per sample per assembler (MEGAHIT, SPAdes, hybridSPAdes)
- Per-assembler QUAST quality summaries
- Merged assembly comparison table quast_summary.tsv

## How to apply

Load preprocessed short-read and long-read FASTQ files per sample and classify each sample by read type (short-read only, long-read only, hybrid) using nf-core/mag's read-type gating in Nextflow. Assemble short-read-only samples de novo with MEGAHIT, with SPAdes as the alternative assembler. When a sample contains long reads, dispatch to hybridSPAdes so long reads are jointly used with the short reads, in addition to the short-read-only assembly mode where applicable. Collect the per-sample contig FASTA files from each assembler and run QUAST on each assembly to generate quality summaries, then merge them into a single comparison table (quast_summary.tsv). Rationale: hybrid assembly combines short- and long-read advantages to yield continuous yet accurate assemblies, and the pipeline's modular DSL2 structure makes the assembler processes independently testable.

## Related tools

- **nf-core/mag** (Nextflow (DSL2) pipeline that gates samples by read type and dispatches each sample to the correct assembly process) — https://github.com/nf-core/mag
- **MEGAHIT** (Primary de novo assembler for short-read-only metagenomic samples) — https://github.com/voutcn/megahit
- **SPAdes** (Alternative de novo assembler for short-read-only samples)
- **hybridSPAdes** (Hybrid assembler invoked when long reads are present, jointly using long and short reads)
- **QUAST** (Summarizes quality features of each assembly and feeds the merged quast_summary.tsv comparison table)

## Examples

```
nf-test test --tag test --profile +docker
```

## Evaluation signals

- Each assembly process (MEGAHIT/SPAdes/hybridSPAdes) executes and exits successfully under nf-test (e.g. 'nf-test test --tag test --profile +docker')
- A contig FASTA file is produced per sample per assembler, with headers matching the expected assembler naming
- A QUAST quality summary is generated for every assembly run
- All per-assembly QUAST summaries merge into a single quast_summary.tsv covering all assemblies
- Hybrid samples produce hybridSPAdes output in addition to (or instead of) the short-read-only assembly, consistent with the read-type gating

## Limitations

- MEGAHIT and SPAdes are short-read assemblers; samples containing long reads must be routed to hybridSPAdes, so incorrect read-type gating would apply the wrong assembler
- Functional annotation, assembly refinement, and bin refinement steps were not yet included in the presented nf-core/mag version 2.1.0 and are listed as future work
- Porechop adapter trimming is skipped for PacBio reads, a filter-step caveat upstream of assembly

## Evidence

- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [intro] Hybrid assembly combines short and long read advantages: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost"
- [intro] Typical reference-independent workflow places assembly after preprocessing: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [methods] Porechop skipped for PacBio reads: "Porechop adapter trimming is skipped for PacBio reads"
- [discussion] Future extensions enabled by DSL2 modular structure: "The modular DSL2 structure efficiently allows future extensions with new tools"
