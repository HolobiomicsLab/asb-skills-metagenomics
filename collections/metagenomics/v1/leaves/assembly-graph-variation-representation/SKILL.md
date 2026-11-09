---
name: assembly-graph-variation-representation
description: Use when assembling metagenomic or mock-community sequence data (e.g.
  the Mock B 12-virus community from SRR961514) where strain-level diversity should
  be preserved and evaluated against alternative assemblers such as MEGAHIT or Ray
  Meta.
when_to_use_negative:
- Input is already-assembled contigs or an existing assembly graph rather than raw/trimmed
  reads
- The goal is isolate-level single-genome assembly without intra-species strain diversity
  to represent
- Only LLVM/clang is available as the compiler — the article states LLVM will not
  compile MetaCortex, so GCC is required
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Two-stage metagenome assembler that imports reads into de Bruijn graphs (CTX
    files), then combines them to output contigs and sequence graphs capturing intra-species
    diversity
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Provides fastq-dump to download the benchmark reads (SRR961514) prior to assembly
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Adapter/quality trimming of reads before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
- name: GCC
  role: Compiler required to build MetaCortex from source (LLVM will not compile it);
    used with 'make MAXK=63 metacortex'
- name: Conda
  role: Alternative installation route for MetaCortex on Linux via the bioconda channel
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:d363d1fe332940d59ddbd0363de60cb7bedd46cb432728298f34c7854369cd32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - Conda
  - GCC
  - MetaCortex
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# assembly-graph-variation-representation

## Summary

Assemble metagenomic/viral reads with MetaCortex so that intra-species diversity is captured as sequence graphs derived from the underlying de Bruijn assembly graph, alongside standard contigs in FASTA.

## When to use

Use when assembling metagenomic or mock-community sequence data (e.g. the Mock B 12-virus community from SRR961514) where strain-level diversity should be preserved and evaluated against alternative assemblers such as MEGAHIT or Ray Meta.

## When NOT to use

- Input is already-assembled contigs or an existing assembly graph rather than raw/trimmed reads
- The goal is isolate-level single-genome assembly without intra-species strain diversity to represent
- Only LLVM/clang is available as the compiler — the article states LLVM will not compile MetaCortex, so GCC is required

## Inputs

- FASTQ reads (e.g. Illumina reads from SRA accession SRR961514, Mock B 12-virus community, Roux 2016)
- Adapter-trimmed reads produced by trim-galore

## Outputs

- contigs.fasta assembled contigs
- CTX intermediate binary de Bruijn graph files
- Sequence graph files capturing local variation along assembled sequences

## How to apply

Download the benchmark reads with the SRA toolkit's fastq-dump command (e.g. accession SRR961514) and trim them with trim-galore before assembly. Clone the MetaCortex repository and compile from source with GCC (LLVM will not compile it) using 'make MAXK=63 metacortex' so the executable bin/metacortex_k63 supports k-mers up to 63 nucleotides; alternatively install via Conda ('conda install -c bioconda metacortex'). Run MetaCortex in its two-stage mode: first, in parallel, import each reads file into a de Bruijn graph and output a CTX file (the intermediate binary format enables parallelisation and lower memory for big assemblies); then combine the separate CTX files to output contigs and sequence graphs. Save the assembled contigs to contigs.fasta and, for benchmarking, evaluate genome fraction and misassembly count with MetaQUAST against comparators such as MEGAHIT (single, min count=20) and Ray Meta (single, k=31).

## Related tools

- **MetaCortex** (Two-stage metagenome assembler that imports reads into de Bruijn graphs (CTX files), then combines them to output contigs and sequence graphs capturing intra-species diversity) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Provides fastq-dump to download the benchmark reads (SRR961514) prior to assembly) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Adapter/quality trimming of reads before assembly) — https://github.com/FelixKrueger/TrimGalore
- **GCC** (Compiler required to build MetaCortex from source (LLVM will not compile it); used with 'make MAXK=63 metacortex')
- **Conda** (Alternative installation route for MetaCortex on Linux via the bioconda channel)

## Examples

```
fastq-dump SRR961514 && trim_galore reads.fastq && git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- On the Mock B 12-virus community, MetaCortex (k=63) should recover ~99.71% genome fraction with a single misassembly when evaluated with MetaQUAST
- MetaQUAST comparison should show larger genome fraction and more contiguous assembly than MEGAHIT (single, min count=20) and Ray Meta (single, k=31)
- The first-stage parallel run must produce one CTX file per input reads file, which the second stage then combines
- Compiled executable bin/metacortex_k63 exists after 'make MAXK=63 metacortex', confirming MAXK supports k-mers up to 63
- Sequence graph output should be produced alongside contigs, indicating local-variation signatures were captured

## Limitations

- LLVM will not compile MetaCortex — Mac users must install GCC and set CC to point to the GCC binary
- MAXK caps the maximum k-mer size at compile time; a different maximum (e.g. MAXK=31 vs 63) requires recompilation
- MetaQUAST-based genome fraction and misassembly benchmarks are tied to the Mock B viral mock community (Roux 2016) and may not transfer directly to other communities

## Evidence

- [abstract] Metacortex captures intra-species diversity via assembly graph sequence graphs: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [intro] Two-stage execution: parallel CTX import then combine: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [methods] Compile with chosen max k-mer value: "make MAXK=63 metacortex"
- [intro] LLVM cannot build MetaCortex; GCC required: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [intro] Benchmark reads retrieved via fastq-dump: "Download the reads using the fastq-dump command from the SRA toolkit::

	fastq-dump SRR961514"
- [abstract] Higher genome coverage and contiguity than other assemblers: "MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers on mock viral communities with high levels of strain-level diversity and"
