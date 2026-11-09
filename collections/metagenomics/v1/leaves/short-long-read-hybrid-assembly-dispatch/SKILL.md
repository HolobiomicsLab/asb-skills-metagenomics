---
name: short-long-read-hybrid-assembly-dispatch
description: Use when preprocessed metagenomic FASTQ samples must be assembled de
  novo and the presence/absence of long reads (Nanopore/PacBio) decides between MEGAHIT/SPAdes
  short-read assembly and hybridSPAdes hybrid assembly.
when_to_use_negative:
- Input is raw reads that still need preprocessing (adapter trimming, quality filtering)
  rather than preprocessed FASTQ — run the preprocessing stage first
- Sample is long-read-only — hybridSPAdes requires both short and long reads; a long-read
  assembler is needed instead
- Task is binning or taxonomic classification of already-assembled contigs/MAGs rather
  than producing the assemblies themselves
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: nf-core/mag
  role: Nextflow DSL2 pipeline that gates samples by read type and dispatches the
    assembly processes
  repo: https://github.com/nf-core/mag
- name: MEGAHIT
  role: de novo assembler for short-read-only metagenomic samples
- name: SPAdes
  role: alternative de novo assembler for short-read-only metagenomic samples
- name: hybridSPAdes
  role: assembler used when both short and long reads are available, jointly using
    both read types
- name: QUAST
  role: summarizes quality features of each assembly and feeds the merged quast_summary.tsv
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:0d89c1d549417891d9bc714d00dc9ff839ba5f63d6f1b3190acad8265689a559
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# short-long-read-hybrid-assembly-dispatch

## Summary

De novo assembly of preprocessed metagenomic reads in nf-core/mag, dispatching short-read-only samples to MEGAHIT or SPAdes and samples that also contain long reads to hybridSPAdes, followed by QUAST-based assembly quality summarization.

## When to use

Use when preprocessed metagenomic FASTQ samples must be assembled de novo and the presence/absence of long reads (Nanopore/PacBio) decides between MEGAHIT/SPAdes short-read assembly and hybridSPAdes hybrid assembly.

## When NOT to use

- Input is raw reads that still need preprocessing (adapter trimming, quality filtering) rather than preprocessed FASTQ — run the preprocessing stage first
- Sample is long-read-only — hybridSPAdes requires both short and long reads; a long-read assembler is needed instead
- Task is binning or taxonomic classification of already-assembled contigs/MAGs rather than producing the assemblies themselves

## Inputs

- preprocessed paired-end short-read FASTQ files per sample
- preprocessed long-read FASTQ files (Oxford Nanopore or PacBio) per sample
- read-type gating configuration (short-read only / long-read only / hybrid) in Nextflow

## Outputs

- contig FASTA assemblies per sample from MEGAHIT, SPAdes, and/or hybridSPAdes
- per-assembler QUAST quality summaries
- merged assembly comparison table quast_summary.tsv

## How to apply

In the Nextflow DSL2 pipeline, first classify each sample by read type (short-read only, long-read only, hybrid) using the pipeline's read-type gating configuration. Short-read-only samples are assembled de novo with MEGAHIT (or SPAdes as the alternative assembler), while samples carrying long reads are routed to hybridSPAdes so the long reads are jointly used with the short reads, run in addition to the short-read-only assembly mode where applicable. The rationale is that hybrid assembly combines the accuracy of short reads with the continuity of long reads at reasonable cost. Collect the contig FASTA files per assembler and sample, run QUAST on each assembly, and merge the QUAST reports into quast_summary.tsv for cross-assembler comparison. Validate the stage by running the pipeline with nf-test so all assembly processes execute and exit successfully.

## Related tools

- **nf-core/mag** (Nextflow DSL2 pipeline that gates samples by read type and dispatches the assembly processes) — https://github.com/nf-core/mag
- **MEGAHIT** (de novo assembler for short-read-only metagenomic samples)
- **SPAdes** (alternative de novo assembler for short-read-only metagenomic samples)
- **hybridSPAdes** (assembler used when both short and long reads are available, jointly using both read types)
- **QUAST** (summarizes quality features of each assembly and feeds the merged quast_summary.tsv comparison)

## Examples

```
nf-test test --tag test --profile +docker
```

## Evaluation signals

- nf-test run of the assembly stage (e.g. 'nf-test test --tag test --profile +docker') exits successfully with all assembly processes executed
- Each sample yields contig FASTA output only from the assembler(s) its read type permits (MEGAHIT/SPAdes for short-read-only; hybridSPAdes additionally for hybrid samples)
- quast_summary.tsv is produced and contains one quality summary row per assembly across all samples
- QUAST metrics are populated (e.g. contig counts, N50-style continuity statistics) and can be compared across assemblers

## Limitations

- At the time of publication, only Muffin among existing metagenomic pipelines supported hybrid assembly, and none supported co-assembly, limiting alternative implementations
- nf-core/mag 2.1.0 did not yet include functional annotation, assembly refinement, or bin refinement steps
- Long-read preprocessing has read-type-specific behavior (e.g. Porechop adapter trimming is skipped for PacBio reads), which must be respected before assembly

## Evidence

- [abstract] Hybrid assembly combines short and long read advantages: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [intro] Rationale for hybrid assembly: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost"
- [intro] nf-core/mag as the vehicle: "Here we introduce nf-core/mag, a Nextflow pipeline for hybrid assembly of metagenomes, binning and taxonomic classification of MAGs"
- [methods] Long-read preprocessing read-type gating: "Porechop adapter trimming is skipped for PacBio reads"
