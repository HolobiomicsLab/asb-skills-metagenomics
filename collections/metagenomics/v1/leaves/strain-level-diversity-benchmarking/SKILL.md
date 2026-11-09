---
name: strain-level-diversity-benchmarking
description: Use when comparing metagenomic assemblers on short-read viral/metagenomic
  data containing closely related strains, where assemblies must be scored against
  reference genomes for genome fraction, misassemblies and per-100 kb error rates.
when_to_use_negative:
- The input is long-read data or non-viral isolate genomes rather than short-read
  metagenomic communities with strain-level diversity — the benchmark is grounded
  in Illumina mock viral communities
- The goal is variant calling or sequence graph exploration only, not end-to-end assembly
  benchmarking against reference genomes
- No reference genomes are available for MetaQUAST-based evaluation, so genome fraction
  and misassembly metrics cannot be computed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaCortex
  role: Target assembler under benchmark; two-stage de Bruijn graph assembly (parallel
    CTX import, then combine CTX files to contigs) at k=95
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Downloads the Mock A Illumina reads with fastq-dump
  repo: https://github.com/ncbi/sra-tools
- name: trim-galore
  role: Adapter-trims reads before assembly to ensure comparable input across assemblers
  repo: https://github.com/FelixKrueger/TrimGalore
- name: GCC
  role: Compiles MetaCortex from source with make MAXK=95; required because LLVM will
    not compile MetaCortex on Macs
- name: Conda
  role: Installs MetaCortex via the bioconda channel as an alternative to source compilation
    (k up to 127)
- name: MetaQUAST
  role: Evaluates all assemblies against the 12 reference virus genomes, producing
    genome fraction, misassemblies, and mismatches/indels per 100 kb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:515825c5642d39706dc5eef9466cdce91b6842d996472b217e3c587913c48b04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - Conda
  - GCC
  - MetaCortex
  - MetaQUAST
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# strain-level-diversity-benchmarking

## Summary

Benchmark the MetaCortex metagenome assembler (k=95) against MEGAHIT, metaSPAdes and Ray Meta on the Mock A 12-virus Illumina community, scoring genome fraction, misassemblies, and mismatch/indel rates per 100 kb with MetaQUAST to quantify performance on high strain-level diversity.

## When to use

Use when comparing metagenomic assemblers on short-read viral/metagenomic data containing closely related strains, where assemblies must be scored against reference genomes for genome fraction, misassemblies and per-100 kb error rates.

## When NOT to use

- The input is long-read data or non-viral isolate genomes rather than short-read metagenomic communities with strain-level diversity — the benchmark is grounded in Illumina mock viral communities
- The goal is variant calling or sequence graph exploration only, not end-to-end assembly benchmarking against reference genomes
- No reference genomes are available for MetaQUAST-based evaluation, so genome fraction and misassembly metrics cannot be computed

## Inputs

- Mock A 12-virus Illumina reads (SRA accession SRR961514, downloaded via fastq-dump)
- 12 reference virus genomes for QUAST evaluation
- Adaptor-trimmed FASTQ reads (trim-galore output) fed to each assembler

## Outputs

- MetaCortex contigs from the two-stage CTX de Bruijn graph assembly (k=95)
- Baseline assemblies from MEGAHIT, metaSPAdes and Ray Meta on identical trimmed reads
- MetaQUAST metrics per assembler: genome fraction (%), misassembly counts, mismatches and indels per 100 kb
- Compiled per-assembler metrics table: mock_a_metacortex_k95_quast_stats.tsv

## How to apply

Download the Mock A 12-virus Illumina reads with fastq-dump from the SRA toolkit (accession SRR961514) and adapter-trim them with trim-galore before assembly. Install MetaCortex either via Conda (conda install -c bioconda metacortex, supporting k up to 127) or by building from source with GCC using make MAXK=95 (compile-time choices 31/63/95/127/160/192); LLVM will not compile MetaCortex on Macs. Run the two-stage assembly: first, in parallel, import each trimmed reads file into a de Bruijn graph and output a CTX file (the intermediate binary format enables parallelisation and lower memory on big assemblies); then combine the separate CTX files to output contigs. Assemble the same trimmed reads with the baseline assemblers (MEGAHIT, metaSPAdes, Ray Meta), then evaluate all assemblies with MetaQUAST against the 12 reference virus genomes to obtain genome fraction, misassembly counts, and mismatches/indels normalised per 100 kb. Compile per-assembler metrics into a stats TSV (mock_a_metacortex_k95_quast_stats.tsv); the MetaCortex k=95 row should reproduce genome fraction 99.89%, zero misassemblies, 4.43 mismatches per 100 kb and 3.44 indels per 100 kb.

## Related tools

- **MetaCortex** (Target assembler under benchmark; two-stage de Bruijn graph assembly (parallel CTX import, then combine CTX files to contigs) at k=95) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Downloads the Mock A Illumina reads with fastq-dump) — https://github.com/ncbi/sra-tools
- **trim-galore** (Adapter-trims reads before assembly to ensure comparable input across assemblers) — https://github.com/FelixKrueger/TrimGalore
- **GCC** (Compiles MetaCortex from source with make MAXK=95; required because LLVM will not compile MetaCortex on Macs)
- **Conda** (Installs MetaCortex via the bioconda channel as an alternative to source compilation (k up to 127))
- **MetaQUAST** (Evaluates all assemblies against the 12 reference virus genomes, producing genome fraction, misassemblies, and mismatches/indels per 100 kb)

## Examples

```
fastq-dump SRR961514; trim_galore SRR961514.fastq; conda install -c bioconda metacortex; make MAXK=95 metacortex
```

## Evaluation signals

- MetaCortex k=95 Mock A row reproduces the reported statistics: genome fraction 99.89%, zero misassemblies, 4.43 mismatches per 100 kb, 3.44 indels per 100 kb
- All assemblers are run on the identical trim-galore-trimmed read set so QUAST metrics are comparable
- MetaQUAST report is generated against all 12 reference virus genomes, with mismatches and indels normalised per 100 kb
- The MetaCortex run completes both stages: per-reads-file CTX outputs exist before CTX files are combined into contigs
- Compiled metrics land in mock_a_metacortex_k95_quast_stats.tsv with one row per assembler

## Limitations

- Conda build of MetaCortex supports only up to kmer 127, so k=160 and k=192 require source compilation with make MAXK
- LLVM cannot compile MetaCortex; Mac users must install GCC and set CC to point to the GCC binary
- Benchmark validation depends on reference genomes being available; metrics are defined only for Mock A-style communities with known references
- Assembler performance claims are tied to mock viral communities with high strain-level diversity and simulated communities; results may not transfer to other data types

## Evidence

- [abstract] MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers: [evidence span withheld — anchor status: not_found]
- [intro] Two-stage assembly process: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [intro] Download reads with fastq-dump: "Download the reads using the fastq-dump command from the SRA toolkit"
- [methods] Conda install path: "conda install -c bioconda metacortex"
- [intro] Build from source with chosen max k-mer: [evidence span withheld — anchor status: not_found]
- [intro] LLVM cannot compile MetaCortex: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
