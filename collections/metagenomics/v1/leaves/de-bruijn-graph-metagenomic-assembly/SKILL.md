---
name: de-bruijn-graph-metagenomic-assembly
description: Use when assembling metagenomic (environmental) FASTQ reads from samples
  with high strain-level diversity — e.g. viral mock communities, HIV lab mixes, or
  gut microbiomes — and both a consensus contig set and per-strain graph paths are
  needed.
when_to_use_negative:
- The input is not raw metagenomic reads but already-assembled contigs or a feature/bin
  table — this skill operates on FASTQ reads via de Bruijn graph construction.
- Only a single-isolate genome with no strain-level variation is targeted; a consensus-only
  isolate assembler is more appropriate than MetaCortex's sequence-graph output.
- Only an LLVM toolchain is available on macOS and the conda build is unsuitable —
  MetaCortex source compilation requires GCC ('LLVM will not compile MetaCortex').
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: De Bruijn graph metagenomic assembler; parallel CTX import then CTX combination
    to output contigs and sequence graphs (GFA) with strain-level variation
  repo: https://github.com/SR-Martin/metacortex.git
- name: trim-galore
  role: Adapter and quality trimming of FASTQ reads prior to graph import
  repo: https://github.com/FelixKrueger/TrimGalore
- name: SRA toolkit
  role: fastq-dump retrieval of benchmark read sets (e.g. SRR961514, ERR2099157)
  repo: https://github.com/ncbi/sra-tools/wiki
- name: GCC
  role: Compiler for building MetaCortex from source with a chosen MAXK; LLVM cannot
    compile the code
- name: Conda
  role: Alternative installation route via bioconda (build fixed at MAXK=127)
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  - task_008
  - task_002
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:4acd8bcb0f3d8480b78010fe638c63f4b9ad5278fa6a777b8ee7fa028a62b6e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - Bowtie2
  - CAMISIM
  - CheckM
  - Conda
  - GCC
  - MEGAHIT
  - MetaBAT2
  - MetaCortex
  - MetaQUAST
  - Ray Meta
  - SRA toolkit
  - dnadiff
  - metaSPAdes
  - trim-galore
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# de-bruijn-graph-metagenomic-assembly

## Summary

Two-stage de Bruijn graph metagenomic assembly with MetaCortex: reads are imported in parallel into CTX binary graph files, then combined to output consensus contigs and sequence graphs (GFA) that retain intra-species variation for strain-level reconstruction.

## When to use

Use when assembling metagenomic (environmental) FASTQ reads from samples with high strain-level diversity — e.g. viral mock communities, HIV lab mixes, or gut microbiomes — and both a consensus contig set and per-strain graph paths are needed.

## When NOT to use

- The input is not raw metagenomic reads but already-assembled contigs or a feature/bin table — this skill operates on FASTQ reads via de Bruijn graph construction.
- Only a single-isolate genome with no strain-level variation is targeted; a consensus-only isolate assembler is more appropriate than MetaCortex's sequence-graph output.
- Only an LLVM toolchain is available on macOS and the conda build is unsuitable — MetaCortex source compilation requires GCC ('LLVM will not compile MetaCortex').

## Inputs

- Trimmed paired-end FASTQ metagenomic reads (e.g. SRR961514 Mock B viral community; ERR2099157 human gut WGS; CAMISIM-simulated 2x150 bp bacterial/viral reads)
- Chosen k-mer size and minimum coverage threshold (k=31–192 depending on compile-time MAXK)
- Reference genomes/strains for MetaQUAST or dnadiff evaluation (optional, for benchmarking)

## Outputs

- Consensus contigs FASTA from the combined CTX graphs (filter to contigs >=500 bp for binning)
- Sequence graph files in GFA format capturing local variation along assembled sequences
- Benchmark tables of genome fraction, contig count, N50, max contig length, and misassembly counts (e.g. hiv_mix_assembly_stats.tsv, simulated-bacterial assembly stats)

## How to apply

First download reads with fastq-dump (e.g. SRR961514, ERR2099157) and trim them with trim-galore before assembly. Build MetaCortex from source with 'make MAXK=63 metacortex' (GCC required — LLVM will not compile it; alternatively 'conda install -c bioconda metacortex', with the conda build fixed at MAXK=127), which fixes the maximum k-mer size at compile time and produces ./bin/metacortex_k63. Run the two-stage workflow: in parallel, import each trimmed reads file into a de Bruijn graph and emit a CTX binary file; then combine the separate CTX files to output contigs and sequence graphs. Choose k per dataset (k=63 with min coverage=5 for simulated bacterial/viral communities, k=127 with delta=0.4 and min coverage=25 for the low-abundance ~2.8% HIV lab mix, k=63 with SW delta=0.8 and min coverage=2 for the human gut sample). Discard contigs <500 bp for downstream binning, and evaluate assemblies against references with MetaQUAST or dnadiff, or walk the GFA sequence graph (e.g. scripts/strain_assembly.py with per-strain BLAST-scored branch selection) to recover strain-specific contigs.

## Related tools

- **MetaCortex** (De Bruijn graph metagenomic assembler; parallel CTX import then CTX combination to output contigs and sequence graphs (GFA) with strain-level variation) — https://github.com/SR-Martin/metacortex.git
- **trim-galore** (Adapter and quality trimming of FASTQ reads prior to graph import) — https://github.com/FelixKrueger/TrimGalore
- **SRA toolkit** (fastq-dump retrieval of benchmark read sets (e.g. SRR961514, ERR2099157)) — https://github.com/ncbi/sra-tools/wiki
- **GCC** (Compiler for building MetaCortex from source with a chosen MAXK; LLVM cannot compile the code)
- **Conda** (Alternative installation route via bioconda (build fixed at MAXK=127))

## Examples

```
make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- MetaQUAST against HIV-1 references: genome fraction ~82.563% across 31 contigs with no misassemblies on the in silico HIV lab mix (k=127, delta=0.4, min coverage=25)
- Mock B viral community (SRR961514): genome fraction ~99.71% with a single misassembly, better than MEGAHIT and Ray Meta
- Simulated bacterial community (CAMISIM, k=63, min coverage=5): genome fraction ~71% vs 27–54% for metaSPAdes/MetaVelvet/Ray Meta; simulated viral community: 98.78% with no misassemblies
- Human gut sample (ERR2099157, k=63, min coverage=2, SW delta=0.8): after dropping contigs <500 bp, expect ~123720 contigs, N50 5399, max contig 879838 bp
- dnadiff of BLAST-scored graph-walk strain-specific contigs vs the original consensus contig: strain-specific paths should show higher average identity and fewer SNPs/indels

## Limitations

- Maximum k-mer size is fixed at compile time via 'make MAXK=...' (values 31–192); the bioconda build is fixed at MAXK=127, so k is a build-time, not run-time, choice.
- LLVM will not compile MetaCortex — Mac users must install GCC and point CC at the GCC binary.
- Ultra-high coverage (e.g. >1000x for S.bongori sim.2) can cause misassembled blocks; subsampling reads to 10% and reassembling halved misassembled blocks for that genome.
- Sequence graphs (GFA) require post-hoc graph walking (e.g. BLAST-scored per-strain branch selection) to convert into strain-specific contigs; consensus contigs alone do not capture the variation.

## Evidence

- [intro] two-stage assembly design: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
- [methods] compile-time MAXK and executable: "MetaCortex can be installed by compiling the source code with GCC"
- [methods] conda install route: "conda install -c bioconda metacortex"
- [abstract] capture variation via assembly graph: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [intro] trimming prerequisite: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [intro] read retrieval: "Download the reads using the fastq-dump command from the SRA toolkit"
- [intro] LLVM incompatibility: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [methods] executable invocation: "./bin/metacortex_k63"
