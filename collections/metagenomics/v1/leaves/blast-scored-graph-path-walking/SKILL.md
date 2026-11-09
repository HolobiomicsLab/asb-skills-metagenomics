---
name: blast-scored-graph-path-walking
description: Use when a metagenomic assembly graph (GFA from MetaCortex MC mode) contains
  unresolved branches from strain-level variation, and consensus contigs must be resolved
  into strain-specific sequences for per-strain validation.
when_to_use_negative:
- The input is a finished single-strain isolate assembly with no strain-level mixture,
  so the sequence graph contains no variation branches to resolve
- No per-strain reference sequences are available to score graph branches with BLAST,
  since the walk depends on BLAST-scored per-strain branch selection
- The goal is a single consensus contig rather than strain-resolved reconstruction
  — the consensus Cortex assembler (cortex_con) path already outputs that
edam_operation: http://edamontology.org/operation_0310
edam_topics: []
tools:
- name: MetaCortex
  role: Metagenome assembler run in MC mode to emit the assembly graph in sequence-graph
    (GFA) format; its scripts/strain_assembly.py performs the BLAST-scored graph walk
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Provides fastq-dump to obtain the HIV lab mix reads used as assembly input
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Trims the reads before assembly into the de Bruijn graph
  repo: https://github.com/FelixKrueger/TrimGalore
- name: dnadiff
  role: Aligns each strain-specific contig to its cognate reference to record average
    identity and SNP/indel counts for validation
- name: GCC
  role: Compiler used to build MetaCortex from source; LLVM will not compile it
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:d467d16274377176dbeada362097221b640dd92940a1cc31dc40938524ec5516
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# BLAST-scored graph path walking

## Summary

A MetaCortex-based technique that walks a sequence graph (GFA) emitted from metagenomic reads, using per-strain BLAST scores to select branch paths and reconstruct strain-specific contigs that capture intra-species variation collapsed in the consensus assembly.

## When to use

Use when a metagenomic assembly graph (GFA from MetaCortex MC mode) contains unresolved branches from strain-level variation, and consensus contigs must be resolved into strain-specific sequences for per-strain validation.

## When NOT to use

- The input is a finished single-strain isolate assembly with no strain-level mixture, so the sequence graph contains no variation branches to resolve
- No per-strain reference sequences are available to score graph branches with BLAST, since the walk depends on BLAST-scored per-strain branch selection
- The goal is a single consensus contig rather than strain-resolved reconstruction — the consensus Cortex assembler (cortex_con) path already outputs that

## Inputs

- FASTQ reads from a metagenomic lab mix (e.g. SRA run SRR961514), adapter-trimmed with trim-galore
- Compiled MetaCortex binary (metacortex_kX, e.g. MAXK=63)
- Per-strain reference sequences used as BLAST guides for branch selection
- Sequence graph in GFA format emitted by MetaCortex MC mode (containing the ~9.2 kb contig and graph elements)

## Outputs

- Five strain-specific contig FASTA files extracted as paths through the sequence graph
- dnadiff comparison table (dnadiff_comparison_table.tsv, reproducing Table 4) with average identity and SNP/indel counts per strain-specific contig vs. the original consensus contig

## How to apply

Obtain metagenomic reads (e.g. fastq-dump SRR961514 from the SRA toolkit), trim them with trim-galore, then compile MetaCortex from source with GCC at an appropriate maximum k-mer size (e.g. `make MAXK=63 metacortex`) — LLVM will not compile MetaCortex. Run the compiled executable (`./bin/metacortex_k63`) in MC mode to import reads into de Bruijn graph CTX files, combine them, and emit the assembly graph in sequence-graph (GFA) format containing the ~9.2 kb contig and its graph elements. Then run scripts/strain_assembly.py to perform the BLAST-scored graph walk: each branch in the sequence graph is scored against per-strain references so that five strain-specific paths are extracted and written as strain-specific contig FASTA files. The rationale is that MetaCortex searches for signatures of local variation along assembled sequences in the underlying assembly graph, so branch selection recovers diversity the consensus contig collapses. Validate by aligning each strain-specific contig to its cognate reference with dnadiff and comparing average identity and SNP/indel counts against the original ~9.2 kb contig.

## Related tools

- **MetaCortex** (Metagenome assembler run in MC mode to emit the assembly graph in sequence-graph (GFA) format; its scripts/strain_assembly.py performs the BLAST-scored graph walk) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Provides fastq-dump to obtain the HIV lab mix reads used as assembly input) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Trims the reads before assembly into the de Bruijn graph) — https://github.com/FelixKrueger/TrimGalore
- **dnadiff** (Aligns each strain-specific contig to its cognate reference to record average identity and SNP/indel counts for validation)
- **GCC** (Compiler used to build MetaCortex from source; LLVM will not compile it)

## Examples

```
make MAXK=63 metacortex && ./bin/metacortex_k63; python scripts/strain_assembly.py
```

## Evaluation signals

- The five strain-specific FASTA files exist after running scripts/strain_assembly.py over the GFA
- The dnadiff table shows higher average identity for each strain-specific contig to its cognate reference than the original ~9.2 kb contig achieves
- Strain-specific contigs contain far fewer SNPs and indels than the original contig, matching the ordering of values in Table 4
- The GFA output contains the ~9.2 kb contig and its graph elements as expected from MC-mode MetaCortex output
- MetaCortex compiles successfully only with GCC (not LLVM) at the chosen MAXK, and ./bin/metacortex_kX executes without error

## Limitations

- LLVM cannot compile MetaCortex; Mac users must install GCC and set CC to the GCC binary
- Branch selection requires per-strain BLAST guides, so accuracy depends on the quality/completeness of the reference strains used for scoring
- Validation was demonstrated on a mock viral community (HIV lab mix, ~9.2 kb genome); behaviour on larger or less-characterized communities is not established by this workflow

## Evidence

- [abstract] MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [methods] Compile with MAXK: "make MAXK=63 metacortex"
- [intro] Assembly graph output in two stages: "Then secondly, to combine the separate CTX files and output contigs and sequence graphs."
- [intro] Build from source requires GCC: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [intro] Trim reads before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [intro] Download reads with fastq-dump: "Download the reads using the fastq-dump command from the SRA toolkit::"
