---
name: assembly-contiguity-quality-assessment
description: Use when short-read and/or long-read metagenome samples have been assembled
  (short-read-only, long-read-only, or hybrid) and you must compare assembly contiguity
  across assemblers or dispatch modes before binning.
when_to_use_negative:
- Input reads are still raw and unpreprocessed — assembly/quality assessment must
  follow read preprocessing in the reference-independent metagenomic workflow
- You need functional or taxonomic annotation of bins rather than assembly contiguity
  metrics — these steps are not covered by this skill
- Only a single isolate assembly exists with no alternative assemblers or dispatch
  modes to compare — a full cross-assembler QUAST comparison table adds little
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: nf-core/mag
  role: Pipeline that gates samples by read type and dispatches short-read-only vs
    hybrid assembly processes; code hosted under the nf-core organization
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine implementing the read-type gating configuration and modular
    DSL2 assembly processes
  repo: https://github.com/nf-core/mag
- name: MEGAHIT
  role: De novo assembler for short-read-only samples; its contig FASTA output is
    a QUAST assessment input
- name: SPAdes
  role: Alternative de novo assembler for short-read-only samples
- name: hybridSPAdes
  role: Hybrid assembler used when samples contain long reads, jointly using long
    reads with short reads
- name: QUAST
  role: Summarizes quality features of each assembly and feeds the merged quast_summary.tsv
    comparison table
- name: nf-test
  role: Used to validate that the assembly processes execute and exit successfully
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:69a6b0496944162327816616cbc3992df93c5cc2406dfaf24c277f2af8b9be6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - MEGAHIT
  - Nextflow
  - QUAST
  - SPAdes
  - hybridSPAdes
  - nf-core/mag
  - nf-test
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# assembly-contiguity-quality-assessment

## Summary

Assesses the contiguity and quality of metagenome de novo assemblies produced by MEGAHIT, SPAdes, and hybridSPAdes in nf-core/mag, using QUAST per-assembly summaries merged into a single quast_summary.tsv comparison table.

## When to use

Use when short-read and/or long-read metagenome samples have been assembled (short-read-only, long-read-only, or hybrid) and you must compare assembly contiguity across assemblers or dispatch modes before binning.

## When NOT to use

- Input reads are still raw and unpreprocessed — assembly/quality assessment must follow read preprocessing in the reference-independent metagenomic workflow
- You need functional or taxonomic annotation of bins rather than assembly contiguity metrics — these steps are not covered by this skill
- Only a single isolate assembly exists with no alternative assemblers or dispatch modes to compare — a full cross-assembler QUAST comparison table adds little

## Inputs

- Preprocessed short-read FASTQ files per sample
- Preprocessed long-read FASTQ files per sample (hybrid samples)
- Contig FASTA files produced per sample by MEGAHIT, SPAdes, or hybridSPAdes

## Outputs

- Per-assembly QUAST quality summary reports
- Merged assembly comparison table quast_summary.tsv covering all assemblies

## How to apply

Load the preprocessed short-read and long-read FASTQ files per sample and classify each sample as short-read only, long-read only, or hybrid via the pipeline's read-type gating in Nextflow. Assemble short-read-only samples with MEGAHIT (or SPAdes as the alternative assembler), and for samples containing long reads dispatch to hybridSPAdes so long reads are used jointly with short reads, in addition to the short-read-only mode where applicable. Collect the resulting per-sample contig FASTA files from each assembler and run QUAST on each assembly to generate per-assembler quality summaries. Merge the QUAST summaries into a single comparison table (quast_summary.tsv) covering all assemblies. The rationale is that hybrid assembly combines short- and long-read advantages to produce continuous and accurate assemblies, so per-assembler QUAST metrics are the evidence base for choosing a downstream assembly. Validate the stage by running the pipeline processes with nf-test so they execute and exit successfully.

## Related tools

- **nf-core/mag** (Pipeline that gates samples by read type and dispatches short-read-only vs hybrid assembly processes; code hosted under the nf-core organization) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine implementing the read-type gating configuration and modular DSL2 assembly processes) — https://github.com/nf-core/mag
- **MEGAHIT** (De novo assembler for short-read-only samples; its contig FASTA output is a QUAST assessment input)
- **SPAdes** (Alternative de novo assembler for short-read-only samples)
- **hybridSPAdes** (Hybrid assembler used when samples contain long reads, jointly using long reads with short reads)
- **QUAST** (Summarizes quality features of each assembly and feeds the merged quast_summary.tsv comparison table)
- **nf-test** (Used to validate that the assembly processes execute and exit successfully) — https://github.com/nf-core/mag

## Examples

```
nf-test test --tag test --profile +docker
```

## Evaluation signals

- quast_summary.tsv exists and contains one quality row per assembly (per sample and per assembler)
- Every sample is classified by read type (short-read only, long-read only, hybrid) and only hybrid samples receive a hybridSPAdes assembly
- Contig FASTA output files are produced for each assembler and each sample without empty/missing files
- The assembly processes execute and exit successfully under nf-test (e.g. 'nf-test test --tag test --profile +docker')
- QUAST summaries are comparable across assemblers, allowing contiguity of hybrid vs short-read-only assemblies to be judged

## Limitations

- Assembly refinement and bin refinement steps are not yet included in the pipeline version presented, so quality assessment cannot cover refinement outcomes
- Co-assembly can change contiguity outcomes since it increases sequencing depth and improves assembly completeness; comparisons must keep assembly mode consistent
- Evaluation depends on nf-core test profiles (e.g. docker) and CAMISIM simulated metagenomic data for exemplary results, which may not reflect all real-sample behavior

## Evidence

- [abstract] Pipeline scope includes assembly and classification: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [abstract] Hybrid assembly improves continuity: "It can optionally combine short and long reads to increase assembly continuity and utilize sample-wise group-information for co-assembly and genome binning."
- [intro] Workflow ordering: preprocessing before assembly: "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [intro] Hybrid assembly rationale for continuous, accurate assemblies: "Hybrid assembly approaches combine the advantages of both short and long reads and thus, can produce continuous and accurate assemblies at reasonable cost"
- [intro] Co-assembly increases depth and completeness: "co-assembly increases sequencing depth and can thus improve assembly completeness"
