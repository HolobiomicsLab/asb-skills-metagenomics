---
name: bacterial-community-read-simulation
description: 'Use when you need to reproduce or extend a simulated-community assembly
  benchmark: you have reference genomes and want to generate in silico metagenomic
  reads (CAMISIM) and rank assemblers by genome fraction and contiguity.'
when_to_use_negative:
- The input is real (non-simulated) metagenomic data without matched reference genomes
  — genome fraction cannot be computed and the benchmark design does not apply
- The community of interest is a mock viral community with high strain-level diversity
  rather than a simulated bacterial community with simulated strains
- You only need to assemble a single isolate dataset — a metagenomic assembler benchmark
  across community members is unnecessary
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: MetaCortex
  role: Primary metagenome assembler under test; run with k=63 and minimum coverage
    5 on the simulated reads
  repo: https://github.com/SR-Martin/metacortex.git
- name: Conda
  role: Installation channel for MetaCortex via the bioconda channel as an alternative
    to source compilation
- name: GCC
  role: Compiler required to build MetaCortex from source with a fixed maximum k-mer
    size (make MAXK=63)
- name: trim-galore
  role: Adapter/quality trimming of the simulated reads before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:2efbb08cb8a84c3f0a7687f22447e874e2cd7ba57e3de4a1d0433aa0410f9e64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - Conda
  - GCC
  - MetaCortex
  - trim-galore
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# bacterial-community-read-simulation

## Summary

Benchmark metagenomic assemblers on a CAMISIM-simulated bacterial community (four MBARC-26 species plus simulated S. bongori and F. pennivorans strains, 14.9 Gb paired-end 2x150 bp reads) by comparing genome fraction recovered by MetaCortex (k=63, min coverage=5) against MEGAHIT, metaSPAdes, MetaVelvet and Ray Meta.

## When to use

Use when you need to reproduce or extend a simulated-community assembly benchmark: you have reference genomes and want to generate in silico metagenomic reads (CAMISIM) and rank assemblers by genome fraction and contiguity.

## When NOT to use

- The input is real (non-simulated) metagenomic data without matched reference genomes — genome fraction cannot be computed and the benchmark design does not apply
- The community of interest is a mock viral community with high strain-level diversity rather than a simulated bacterial community with simulated strains
- You only need to assemble a single isolate dataset — a metagenomic assembler benchmark across community members is unnecessary

## Inputs

- Reference genomes of four MBARC-26 bacterial species plus simulated S. bongori and F. pennivorans strains
- CAMISIM-simulated metagenomic reads (14.9 Gb, paired-end 2x150 bp FASTQ)
- MetaCortex source code (github.com/SR-Martin/metacortex) or bioconda package

## Outputs

- MetaCortex assembly (contigs/sequence graphs) produced with k=63 and minimum coverage 5
- Assemblies from MEGAHIT, metaSPAdes, MetaVelvet and Ray Meta on the identical read set
- Simulated-bacterial assembly statistics table with genome fraction (percentage of reference bases covered) and contiguity metrics per assembler

## How to apply

Regenerate the CAMISIM simulation of the bacterial community containing four MBARC-26 species plus simulated Salmonella bongori and Fontibacter? pennivorans strains, producing 14.9 Gb of paired-end 2x150 bp reads. Install MetaCortex either via 'conda install -c bioconda metacortex' or by compiling from the cloned GitHub repository with 'make MAXK=63 metacortex' (GCC required; LLVM will not compile it), then run ./bin/metacortex_k63. Trim reads with trim-galore, run MetaCortex with k=63 and minimum coverage 5, and assemble the identical read set with MEGAHIT, metaSPAdes, MetaVelvet and Ray Meta using default recommended parameters. Evaluate every assembly against the reference genomes by computing genome fraction (percentage of reference bases covered) and contiguity metrics, and collate per-assembler metrics into the simulated-bacterial assembly statistics table. The expected outcome is that MetaCortex recovers the highest genome fraction (71%) while the baseline assemblers fall in the 27–54% range.

## Related tools

- **MetaCortex** (Primary metagenome assembler under test; run with k=63 and minimum coverage 5 on the simulated reads) — https://github.com/SR-Martin/metacortex.git
- **Conda** (Installation channel for MetaCortex via the bioconda channel as an alternative to source compilation)
- **GCC** (Compiler required to build MetaCortex from source with a fixed maximum k-mer size (make MAXK=63))
- **trim-galore** (Adapter/quality trimming of the simulated reads before assembly) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- MetaCortex reproduces the reported genome fraction of 71% on the simulated bacterial community
- Baseline assemblers (metaSPAdes, MetaVelvet, Ray Meta) fall in the reported 27–54% genome fraction range
- Genome fraction is computed as percentage of reference bases covered for every assembly against the same reference genomes
- All assemblers were run on the identical trimmed read set (14.9 Gb paired-end 2x150 bp) so metrics are directly comparable
- Contiguity metrics are reported alongside genome fraction in the collated statistics table

## Limitations

- Results depend on CAMISIM simulation parameters; the benchmark community is fixed (four MBARC-26 species plus simulated S. bongori and F. pennivorans strains, 14.9 Gb of 2x150 bp reads)
- MetaCortex must be compiled with a maximum k-mer size fixed at build time (e.g. MAXK=63); LLVM will not compile it, so Mac users must install GCC and set CC to the GCC binary
- Baseline assemblers were run with default recommended parameters, so the comparison does not explore per-assembler parameter tuning

## Evidence

- [abstract] MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers on mock viral communities with high strain-level diversity and: [evidence span withheld — anchor status: not_found]
- [intro] LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [methods] make MAXK=63 metacortex: "MetaCortex can be installed by compiling the source code with GCC"
- [methods] conda install -c bioconda metacortex: "conda install -c bioconda metacortex"
- [intro] we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly: "and we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
