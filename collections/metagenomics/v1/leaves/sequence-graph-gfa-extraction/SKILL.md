---
name: sequence-graph-gfa-extraction
description: Use when a metagenomic sample contains closely related strains (e.g.
  the HIV lab mix) and the goal is to recover per-strain sequences from a de Bruijn
  assembly graph rather than settling for a single consensus contig.
when_to_use_negative:
- When the goal is a single consensus assembly for coverage/contiguity benchmarking
  rather than per-strain sequence recovery — consensus assemblers are sufficient and
  strain graph walking adds unnecessary complexity
- When the sample has low strain-level diversity and no local variation signatures
  exist in the assembly graph, leaving no graph branches to resolve
- When no per-strain reference genomes are available for BLAST-scored branch selection
  or dnadiff validation
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Builds the de Bruijn sequence graph and outputs contigs and sequence graphs
    in GFA format; source of the graph walked for strain-specific paths
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Obtains the input reads via fastq-dump (e.g. run SRR961514)
  repo: https://github.com/ncbi/sra-tools
- name: trim-galore
  role: Trims reads before assembly so the de Bruijn graph is built from clean reads
  repo: https://github.com/FelixKrueger/TrimGalore
- name: dnadiff
  role: Aligns strain-specific contigs to cognate strain references to record average
    identity and SNP/indel counts for validation
- name: GCC
  role: Compiles MetaCortex from source (LLVM will not compile MetaCortex)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:353a3f6312a7963db9ac9f258f02349222ad4d70534d01197b0aba1d013cfeb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - GCC
  - MetaCortex
  - SRA toolkit
  - dnadiff
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# sequence-graph-gfa-extraction

## Summary

Reconstructs strain-specific contigs from MetaCortex's sequence-graph (GFA) output by performing per-strain BLAST-scored graph walks, capturing intra-species variation that consensus metagenomic assembly collapses into a single contig. Applied with dnadiff to validate that strain-specific paths are more accurate per strain than the consensus contig.

## When to use

Use when a metagenomic sample contains closely related strains (e.g. the HIV lab mix) and the goal is to recover per-strain sequences from a de Bruijn assembly graph rather than settling for a single consensus contig.

## When NOT to use

- When the goal is a single consensus assembly for coverage/contiguity benchmarking rather than per-strain sequence recovery — consensus assemblers are sufficient and strain graph walking adds unnecessary complexity
- When the sample has low strain-level diversity and no local variation signatures exist in the assembly graph, leaving no graph branches to resolve
- When no per-strain reference genomes are available for BLAST-scored branch selection or dnadiff validation

## Inputs

- Metagenomic sequencing reads (e.g. SRA run SRR961514 from the HIV lab mix, fetched with fastq-dump)
- Trimmed FASTQ reads (trim-galore output) for de Bruijn graph construction
- Per-strain reference genomes (e.g. reference HIV strains) for BLAST scoring and dnadiff validation

## Outputs

- Sequence-graph (GFA) format assembly graph containing contigs and their graph elements
- Strain-specific contig FASTA files (one per strain-specific graph walk)
- dnadiff comparison table (e.g. dnadiff_comparison_table.tsv) with average identity and SNP/indel counts per strain-specific contig vs. the original consensus contig

## How to apply

Obtain reads (e.g. SRR961514 via fastq-dump from the SRA toolkit) and trim them with trim-galore before assembly. Compile MetaCortex from source with GCC at an appropriate maximum k-mer size (e.g. make MAXK=63 metacortex) and run ./bin/metacortex_kX in MC mode to emit the assembly graph in sequence-graph (GFA) format containing the ~9.2 kb contig and its graph elements. Run scripts/strain_assembly.py to perform a per-strain BLAST-scored walk over the sequence graph, extracting strain-specific paths (five in the HIV lab mix case) written as strain-specific contig FASTA files. Validate each path by aligning it to its cognate strain reference with dnadiff and recording average identity plus SNP/indel counts, comparing against the original consensus contig. The rationale is that MetaCortex captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph, so variation conserved only per-strain remains explorable as graph elements rather than being collapsed.

## Related tools

- **MetaCortex** (Builds the de Bruijn sequence graph and outputs contigs and sequence graphs in GFA format; source of the graph walked for strain-specific paths) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Obtains the input reads via fastq-dump (e.g. run SRR961514)) — https://github.com/ncbi/sra-tools
- **trim-galore** (Trims reads before assembly so the de Bruijn graph is built from clean reads) — https://github.com/FelixKrueger/TrimGalore
- **dnadiff** (Aligns strain-specific contigs to cognate strain references to record average identity and SNP/indel counts for validation)
- **GCC** (Compiles MetaCortex from source (LLVM will not compile MetaCortex))

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- The five strain-specific FASTA files exist and each corresponds to a distinct graph walk over the GFA sequence graph
- dnadiff shows higher average alignment identity for each strain-specific contig to its cognate reference than for the original single ~9.2 kb consensus contig
- dnadiff shows fewer SNPs and indels for strain-specific contigs than for the consensus contig, reproducing the ordering of values reported in Table 4
- The MetaCortex run in MC mode emits a sequence-graph (GFA) file containing the ~9.2 kb contig and its graph elements
- Compilation completes with the chosen MAXK (e.g. 63) and the bin executable ./bin/metacortex_kX runs in MC mode without error

## Limitations

- LLVM will not compile MetaCortex, so Mac users must install GCC and set CC to point to the GCC binary
- Validation of strain-specific paths requires per-strain reference genomes; the accuracy gain is only demonstrated against cognate references
- The maximum k-mer size must be chosen at compile time (MAXK), which constrains the k-mer resolution of the sequence graph

## Evidence

- [abstract] MetaCortex captures intra-species diversity by searching for signatures of local variation along assembled sequences in the assembly graph and outputs sequences in sequence graph format: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph and outputting these"
- [intro] MetaCortex is built from source with a chosen MAXK and runs two-stage assembly to output contigs and sequence graphs: "Then secondly, to combine the separate CTX files and output contigs and sequence graphs."
- [intro] LLVM cannot compile MetaCortex; GCC must be used: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [methods] MetaCortex is installed by compiling source with a chosen maximum k-mer size: "make MAXK=63 metacortex"
- [abstract] MetaCortex source and versioned release are freely available: "Source code is freely available to download from https://github.com/SR-Martin/metacortex, is implemented in C and supported on MacOS and Linux."
- [intro] Reads are downloaded with fastq-dump from the SRA toolkit and trimmed with trim-galore before assembly: "Download the reads using the fastq-dump command from the SRA toolkit"
