---
name: olc-assembly-pipeline-handling
description: Use when the input is interleaved FASTQ metagenomic short-read data (e.g.
  all_reads.fq from an Illumina MiSeq run) and the goal is de novo, strain-resolved
  assembly without a reference genome.
when_to_use_negative:
- Input is long third-generation reads — the paper notes the high error rate of third-generation
  sequencing reads introduces unresolved issues, and the method targets short Illumina-type
  reads
- A high-quality reference genome is available and reference-assisted assembly is
  acceptable — StrainXpress is a de novo method designed to avoid reference bias
- Only a single isolate genome needs assembly without strain mixture — general assemblers
  like SPAdes or MEGAHIT may suffice
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Main OLC assembler executed via scripts/strainxpress.py; performs read clustering,
    local assembly and global assembly
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: 'Overlap stage: computes the read-pair overlap-score file consumed by the
    single-linkage clustering step'
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress installation
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:c05be7ce48a42e9539ee4f8ebee1ef3cc8e9ea912638ad0d60385cb0b561afa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# olc-assembly-pipeline-handling

## Summary

Runs the StrainXpress divide-and-conquer strain-aware metagenome assembly pipeline, whose OLC paradigm passes through three stages — 'Clustering Reads', 'Local Assembly' and 'Global Assembly' — to reconstruct strain-specific genomes from short Illumina-style reads at strain resolution.

## When to use

Use when the input is interleaved FASTQ metagenomic short-read data (e.g. all_reads.fq from an Illumina MiSeq run) and the goal is de novo, strain-resolved assembly without a reference genome.

## When NOT to use

- Input is long third-generation reads — the paper notes the high error rate of third-generation sequencing reads introduces unresolved issues, and the method targets short Illumina-type reads
- A high-quality reference genome is available and reference-assisted assembly is acceptable — StrainXpress is a de novo method designed to avoid reference bias
- Only a single isolate genome needs assembly without strain mixture — general assemblers like SPAdes or MEGAHIT may suffice

## Inputs

- Interleaved paired-end FASTQ file of metagenomic short reads (all_reads.fq, records like @S0R0/1 and @S0R0/2)

## Outputs

- Strain-resolved assembled contigs written to the stageb folder
- Per-cluster read assignments (cluster_assignments.tsv mapping read IDs to cluster IDs) from the Clustering Reads stage

## How to apply

Install StrainXpress via Conda (python 3.6, scipy, pandas, minimap2 from bioconda) plus g++ >=5.5.0 with boost libraries, then clone and compile with `sh install.sh`. Provide a single interleaved FASTQ file (all_reads.fq) with paired records like @S0R0/1 and @S0R0/2 as input to scripts/strainxpress.py; StrainXpress computes read overlaps with minimap2 as the overlap stage of its overlap-layout-consensus (OLC) workflow. The pipeline proceeds through its three stages: single-linkage read clustering (sorted overlap scores, merged top-to-bottom to yield species-coherent clusters), Local Assembly, and Global Assembly. For large data sets use the -fast flag to switch to the fast cluster method, as recommended in the README. Results are written to the stageb folder, from which strain-specific contigs are collected and quality can be evaluated with Quast/metaQuast.

## Related tools

- **StrainXpress** (Main OLC assembler executed via scripts/strainxpress.py; performs read clustering, local assembly and global assembly) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Overlap stage: computes the read-pair overlap-score file consumed by the single-linkage clustering step) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress installation) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- stageb output folder exists and contains assembled strain-specific contigs
- cluster_assignments.tsv is well-formed TSV with exactly one assignment row per input read and non-empty cluster IDs
- Number of clusters is species-coherent (reads from the same species fall into the same cluster); paper reports StrainXpress yields 899 clusters versus 568 for the earlier method
- Assembly quality metrics via Quast/metaQuast show high genome fraction and identity (e.g. 93.45% genome fraction vs 74.74% for SPAdes on low-complexity 2x250 bp data; 99.93% identity)

## Limitations

- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs that span entire genomes
- High error rate of third-generation sequencing reads remains unresolved
- Pipeline is built for linux-based systems and python3 only (python 3.6, scipy, pandas, minimap2, g++ >=5.5.0 with boost); interleaved FASTQ is the only accepted input layout
- StrainXpress clustering is 46 times faster than earlier work and Local Assembly 8 times faster, but very large data sets should still use the -fast clustering flag

## Evidence

- [results] Stages of the StrainXpress pipeline: "the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
- [readme] OLC paradigm description: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data"
- [readme] Input format requirement: "The input file must be interleaved FASTQ and format like below"
- [readme] Fast clustering for big data sets: "When a data set is big, we recommend to use the fast cluster method"
- [discussion] De novo strain-resolved assembly goal: "we have presented StrainXpress, a de novo method that assembles individual genomes from metagenomes at strain resolution"
- [abstract] Comprehensive solution positioning: "Here, we present StrainXpress, as a comprehensive solution to the problem of strain aware metagenome assembly from next-generation sequencing reads."
