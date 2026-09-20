---
name: de-bruijn-graph-kmer-parameterization
description: Use when assembling metagenomic (environmental) FASTQ reads with MetaCortex
  and a maximum k-mer size must be chosen and baked in at compile time before building
  the de Bruijn graph.
when_to_use_negative:
- 'If you need to explore multiple k-mer values post-hoc on the same installation:
  the max k-mer is a compile-time choice, so you must recompile (or install a different
  Conda build) rather than pass k as a runtime flag'
- 'If the required kmer exceeds the Conda build limit of 127: compile from source
  with make MAXK=160 or 192 instead of using the bioconda package'
- 'If the data are not short-read metagenomic sequence data: MetaCortex is built on
  the consensus Cortex assembler (cortex_con) for environmental sequence data, not
  for other data modalities'
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: The assembler whose de Bruijn graph construction is parameterized by the compile-time
    MAXK k-mer value; built from source or installed via Conda
  repo: https://github.com/SR-Martin/metacortex.git
- name: GCC
  role: Compiler required to build MetaCortex with the chosen MAXK value (LLVM will
    not compile MetaCortex)
- name: Conda
  role: Alternative installation route via the bioconda channel, supporting kmers
    up to 127
- name: SRA toolkit
  role: Provides fastq-dump to obtain the reads (e.g. SRR961514) that the parameterized
    de Bruijn graph build will consume
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Adapter-trims reads before they are imported into the de Bruijn graph
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:351b1ee23c741daea9b5558329332511cf625d25eafac4789a3da66daecfff2e
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# de-bruijn-graph-kmer-parameterization

## Summary

Selects and compiles MetaCortex with an appropriate maximum k-mer size (one of 31/63/95/127/160/192) so the de Bruijn graph construction captures strain-level variation in metagenomic viral read data. The k-mer choice is a compile-time parameter, so it must be fixed before assembly.

## When to use

Use when assembling metagenomic (environmental) FASTQ reads with MetaCortex and a maximum k-mer size must be chosen and baked in at compile time before building the de Bruijn graph.

## When NOT to use

- If you need to explore multiple k-mer values post-hoc on the same installation: the max k-mer is a compile-time choice, so you must recompile (or install a different Conda build) rather than pass k as a runtime flag
- If the required kmer exceeds the Conda build limit of 127: compile from source with make MAXK=160 or 192 instead of using the bioconda package
- If the data are not short-read metagenomic sequence data: MetaCortex is built on the consensus Cortex assembler (cortex_con) for environmental sequence data, not for other data modalities

## Inputs

- Metagenomic Illumina FASTQ reads (e.g. Mock A 12-virus community reads, SRA accession SRR961514, downloaded with fastq-dump)
- Adapter-trimmed FASTQ reads (trimmed with trim-galore before graph construction)

## Outputs

- Compiled MetaCortex executable (e.g. ./bin/metacortex_k95) built for the chosen maximum k-mer value
- CTX binary files encoding the de Bruijn graph per input reads file
- Assembled contigs and sequence graph (GFA-format) outputs from the combined CTX files

## How to apply

Clone the MetaCortex repository (github.com/SR-Martin/metacortex) and compile the source with GCC using make MAXK=<k>, where k is one of 31/63/95/127/160/192; for the Mock A 12-virus benchmark k=95 was used and reproduces 99.89% genome fraction with zero misassemblies. Note that the Conda build (conda install -c bioconda metacortex) supports up to kmer 127, so k=95 is covered by either route; LLVM cannot be used, so Mac users must set CC to a GCC binary. After compilation, run the two-stage assembly: in parallel, import each reads file into a de Bruijn graph and output a CTX file, then combine the separate CTX files to output contigs and sequence graphs. The rationale for the parameterization step is that the max kmer value is fixed at build time and cannot be changed afterwards, so it must match the strain-level diversity and read length of the target community before any reads are processed.

## Related tools

- **MetaCortex** (The assembler whose de Bruijn graph construction is parameterized by the compile-time MAXK k-mer value; built from source or installed via Conda) — https://github.com/SR-Martin/metacortex.git
- **GCC** (Compiler required to build MetaCortex with the chosen MAXK value (LLVM will not compile MetaCortex))
- **Conda** (Alternative installation route via the bioconda channel, supporting kmers up to 127)
- **SRA toolkit** (Provides fastq-dump to obtain the reads (e.g. SRR961514) that the parameterized de Bruijn graph build will consume) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Adapter-trims reads before they are imported into the de Bruijn graph) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=95 metacortex
```

## Evaluation signals

- The compiled binary in ./bin matches the requested kmer (e.g. ./bin/metacortex_k95 after make MAXK=95)
- The chosen kmer is within the allowed set 31/63/95/127/160/192, and within 127 if installed via the bioconda Conda build
- On the Mock A 12-virus read set with k=95, MetaQUAST reports genome fraction 99.89% with zero misassemblies, ~4.43 mismatches and ~3.44 indels per 100 kb
- The two-stage run produced one CTX file per input reads file, and the second stage successfully combined the separate CTX files into contigs and sequence graphs
- Compilation succeeded with GCC (not LLVM); on macOS CC points to the GCC binary

## Limitations

- The maximum k-mer value is fixed at compile time (one of 31/63/95/127/160/192); changing it requires recompilation
- The Conda installation supports up to kmer 127, so larger k values require building from source
- LLVM will not compile MetaCortex, so Mac users must install GCC and set CC accordingly
- The k=95 result (99.89% genome fraction, no misassemblies) was validated on the Mock A 12-virus mock community; other communities may favor different k values

## Evidence

- [intro] two-stage CTX assembly: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [methods] compile-time kmer build: [evidence span withheld — anchor status: not_found]
- [methods] conda install limit: "conda install -c bioconda metacortex"
- [intro] GCC requirement: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [intro] trim before assembly: "trim the reads using adapter-trimming software (e.g. trim-galore)"
