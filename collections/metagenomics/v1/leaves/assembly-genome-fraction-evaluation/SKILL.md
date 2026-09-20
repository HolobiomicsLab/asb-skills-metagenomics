---
name: assembly-genome-fraction-evaluation
description: Use when comparing de novo metagenomic assemblers on a community with
  known reference genomes and you need quantitative genome recovery (genome fraction
  %) and contiguity metrics to rank assemblers.
when_to_use_negative:
- There are no reference genomes available, so genome fraction (percentage of reference
  bases covered) cannot be computed — use reference-free quality metrics instead
- The read set derives from a single isolate rather than a mixed community — the metagenomic
  assembler comparison framing (and strain-level diversity evaluation) does not apply
- You only need to run one assembler and are not comparing it against baselines such
  as MEGAHIT, metaSPAdes, MetaVelvet or Ray Meta
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0080
tools:
- name: MetaCortex
  role: Assembler under test; built with MAXK=63 and run as ./bin/metacortex_k63 with
    k=63 and minimum coverage 5; installed via conda (bioconda) or compiled from source
    with GCC
  repo: https://github.com/SR-Martin/metacortex
- name: trim-galore
  role: Adapter/quality trimming of the simulated reads prior to assembly with all
    compared assemblers
  repo: https://github.com/FelixKrueger/TrimGalore
- name: GCC
  role: Compiler required to build MetaCortex from source when not installing via
    Conda
- name: Conda
  role: Alternative installation channel for MetaCortex (conda install -c bioconda
    metacortex)
- name: SRA toolkit
  role: Obtains sequencing reads (fastq-dump) for assembly evaluation workflows
  repo: https://github.com/ncbi/sra-tools/wiki
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:13ef997570179bf86625f2a5a39170dcea3d33adc73fd65b085049b59d4b077a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex
  tools:
  - Conda
  - GCC
  - MetaCortex
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# assembly-genome-fraction-evaluation

## Summary

Benchmark metagenomic assemblers by computing genome fraction (percentage of reference bases covered) and contiguity metrics against known reference genomes, as done for MetaCortex versus MEGAHIT, metaSPAdes, MetaVelvet and Ray Meta on a CAMISIM-simulated bacterial community (14.9 Gb paired-end 2x150 bp reads).

## When to use

Use when comparing de novo metagenomic assemblers on a community with known reference genomes and you need quantitative genome recovery (genome fraction %) and contiguity metrics to rank assemblers.

## When NOT to use

- There are no reference genomes available, so genome fraction (percentage of reference bases covered) cannot be computed — use reference-free quality metrics instead
- The read set derives from a single isolate rather than a mixed community — the metagenomic assembler comparison framing (and strain-level diversity evaluation) does not apply
- You only need to run one assembler and are not comparing it against baselines such as MEGAHIT, metaSPAdes, MetaVelvet or Ray Meta

## Inputs

- Paired-end 2x150 bp metagenomic FASTQ reads (~14.9 Gb, CAMISIM-simulated bacterial community)
- Trimmed reads (trim-galore output) from the same simulated read set used for every assembler
- Reference genome sequences (FASTA) for the four MBARC-26 species and the simulated S. bongori and F. pennivorans strains

## Outputs

- Contig assemblies (FASTA) from each assembler (MetaCortex k=63, MEGAHIT, metaSPAdes, MetaVelvet, Ray Meta)
- Per-assembler genome fraction values (percentage of reference bases covered)
- Contiguity metrics per assembly
- Collated simulated-bacterial assembly statistics table

## How to apply

Assemble the identical trimmed read set (e.g. trim-galore-processed CAMISIM-simulated reads of four MBARC-26 species plus simulated S. bongori and F. pennivorans strains) with each assembler under fair parameters — for MetaCortex build with 'make MAXK=63 metacortex' (or 'conda install -c bioconda metacortex') and run './bin/metacortex_k63' with k=63 and a minimum coverage of 5; run MEGAHIT, metaSPAdes, MetaVelvet and Ray Meta with default recommended parameters. Then align each assembly's contigs to the reference genomes and compute genome fraction, i.e. the percentage of reference bases covered, alongside contiguity metrics. Collate the per-assembler metrics into a single statistics table and rank assemblers by genome fraction. Validation criterion from this benchmark: MetaCortex should reach ~71% genome fraction while baseline assemblers fall in the reported 27–54% range on this simulated community.

## Related tools

- **MetaCortex** (Assembler under test; built with MAXK=63 and run as ./bin/metacortex_k63 with k=63 and minimum coverage 5; installed via conda (bioconda) or compiled from source with GCC) — https://github.com/SR-Martin/metacortex
- **trim-galore** (Adapter/quality trimming of the simulated reads prior to assembly with all compared assemblers) — https://github.com/FelixKrueger/TrimGalore
- **GCC** (Compiler required to build MetaCortex from source when not installing via Conda)
- **Conda** (Alternative installation channel for MetaCortex (conda install -c bioconda metacortex))
- **SRA toolkit** (Obtains sequencing reads (fastq-dump) for assembly evaluation workflows) — https://github.com/ncbi/sra-tools/wiki

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- MetaCortex (k=63, min coverage=5) reproduces the reported 71% genome fraction on the CAMISIM-simulated bacterial community
- Baseline assemblers (metaSPAdes, MetaVelvet, Ray Meta) fall within the reported 27–54% genome fraction range on the same community
- Every assembler received the identical trimmed read set, so cross-assembler genome fraction differences reflect assembly, not input, differences
- Genome fraction is reported as percentage of reference bases covered per reference genome, alongside contiguity metrics
- Assembled contigs align to the expected MBARC-26, S. bongori and F. pennivorans reference genomes

## Limitations

- Genome-fraction evaluation requires reference genomes; findings are specific to the tested communities (mock viral communities with high strain-level diversity and simulated communities with simulated strains)
- MetaCortex must be compiled with the maximum k-mer size fixed at build time (e.g. MAXK=63); LLVM will not compile MetaCortex, so Mac users need GCC
- MetaCortex runs in two stages (parallel import of reads into CTX de Bruijn graph files, then combining CTX files), which affects workflow orchestration

## Evidence

- [abstract] MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers: [evidence span withheld — anchor status: not_found]
- [results] Genome fraction is computed against reference genomes: [evidence span withheld — anchor status: not_found]
- [methods] Compile with a chosen maximum k-mer size: "MetaCortex can be installed by compiling the source code with GCC"
- [intro] Trim reads before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
