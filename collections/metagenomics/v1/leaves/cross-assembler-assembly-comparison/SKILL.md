---
name: cross-assembler-assembly-comparison
description: Use when evaluating a new or candidate metagenomic assembler and you
  need reproducible genome fraction and misassembly comparisons against established
  assemblers on the same simulated or mock community reads.
when_to_use_negative:
- The input is a single-isolate genome with no strain-level variation — the comparison
  targets high strain-level diversity communities where MetaCortex's variation-capturing
  design matters
- You need a functional annotation or taxonomic profiling comparison rather than assembly
  quality (genome fraction / misassemblies) metrics
- The reads are long-read or hybrid data — the workflow is built around 2x150 bp paired-end
  short-read simulations
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Candidate assembler under benchmarking; compiled with MAXK=63 and run with
    min coverage threshold 5 on the simulated reads
  repo: https://github.com/SR-Martin/metacortex.git
- name: MEGAHIT
  role: Baseline competitor assembler run with recommended default settings on the
    identical read set
- name: metaSPAdes
  role: Baseline competitor assembler run with recommended default settings on the
    identical read set
- name: Ray Meta
  role: Baseline competitor assembler run with recommended default settings on the
    identical read set
- name: CAMISIM
  role: Regenerates the simulated viral community (reads from six real virus species
    plus four simulated Influenza B strains) per the Methods simulation parameters
- name: GCC
  role: Compiler used to build MetaCortex from source (LLVM will not compile MetaCortex)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:57c9c2ece950e3ae1355f54f3e66900b3e0e545cad015caa47650bbb3f34e91d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - CAMISIM
  - GCC
  - MEGAHIT
  - MetaCortex
  - Ray Meta
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-assembler-assembly-comparison

## Summary

Benchmark a metagenomic assembler (MetaCortex) head-to-head against MEGAHIT, metaSPAdes and Ray Meta by running all four on the identical read set and comparing per-genome assembly metrics (genome fraction, misassemblies) against reference genomes.

## When to use

Use when evaluating a new or candidate metagenomic assembler and you need reproducible genome fraction and misassembly comparisons against established assemblers on the same simulated or mock community reads.

## When NOT to use

- The input is a single-isolate genome with no strain-level variation — the comparison targets high strain-level diversity communities where MetaCortex's variation-capturing design matters
- You need a functional annotation or taxonomic profiling comparison rather than assembly quality (genome fraction / misassemblies) metrics
- The reads are long-read or hybrid data — the workflow is built around 2x150 bp paired-end short-read simulations

## Inputs

- Simulated paired-end FASTQ reads (2x150 bp, 15 Gb) from a CAMISIM-simulated viral community
- Reference genome FASTA files for the community members (six real virus species plus four simulated Influenza B strains)
- Compiler toolchain (GCC) and CAMISIM for regenerating the simulation
- Competitor assembler executables: MEGAHIT, metaSPAdes, Ray Meta

## Outputs

- Per-assembler contigs and sequence graph outputs (GFA-style sequence graphs for MetaCortex)
- Per-assembler, per-genome genome fraction and misassembly counts
- Summary statistics table, e.g. table5_simulated_viral_assembly_stats.csv

## How to apply

Assemble one identical read set (e.g. 15 Gb of 2x150 bp paired-end reads from a CAMISIM-simulated viral community of six real virus species plus four simulated Influenza B strains) with each assembler. Build MetaCortex from source with a chosen max k-mer (make MAXK=63 metacortex) and run the resulting metacortex_k63 executable with a minimum coverage threshold of 5, while running MEGAHIT, metaSPAdes and Ray Meta with their recommended default settings. Evaluate all four assemblies against the reference genomes with an alignment-based evaluator, computing genome fraction and misassembly counts per assembler and per genome. Compile per-assembler statistics into a summary table (e.g. table5_simulated_viral_assembly_stats.csv) and validate against the reported benchmarks: MetaCortex 98.78% overall genome fraction with individual genome fractions 95.27–99.91% and no misassemblies, with Ray Meta/metaSPAdes in the 77–93% genome fraction range. The rationale is that only identical inputs and controlled parameters (k=63, min coverage 5 for MetaCortex) isolate assembler quality as the source of metric differences.

## Related tools

- **MetaCortex** (Candidate assembler under benchmarking; compiled with MAXK=63 and run with min coverage threshold 5 on the simulated reads) — https://github.com/SR-Martin/metacortex.git
- **MEGAHIT** (Baseline competitor assembler run with recommended default settings on the identical read set)
- **metaSPAdes** (Baseline competitor assembler run with recommended default settings on the identical read set)
- **Ray Meta** (Baseline competitor assembler run with recommended default settings on the identical read set)
- **CAMISIM** (Regenerates the simulated viral community (reads from six real virus species plus four simulated Influenza B strains) per the Methods simulation parameters)
- **GCC** (Compiler used to build MetaCortex from source (LLVM will not compile MetaCortex))

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git; cd metacortex && make MAXK=63 metacortex; ./bin/metacortex_k63
```

## Evaluation signals

- MetaCortex must reach 98.78% overall genome fraction on the CAMISIM-simulated viral community, with per-genome fractions spanning 95.27% to 99.91%
- MetaCortex assemblies must report zero misassemblies against the reference genomes
- Ray Meta and metaSPAdes baseline assemblies should fall in the 77–93% genome fraction range on the same reads
- All four assemblers must consume the byte-identical read set (same CAMISIM run, 15 Gb of 2x150 bp paired-end reads) to make the comparison valid
- The compiled per-assembler statistics must match the reported Table 5 values to count as a successful reproduction

## Limitations

- MetaCortex must be compiled with GCC (LLVM will not compile it), so Mac users must install GCC and set CC to point to the GCC binary
- The k-mer size is fixed at compile time via MAXK (e.g. MAXK=63 producing the metacortex_k63 executable), so changing k requires recompilation
- The benchmark reflects simulated viral communities with high strain-level diversity; results may not transfer to low-diversity or non-viral communities

## Evidence

- [abstract] MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers on mock viral communities: "MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers on mock viral communities with high levels of strain-level diversity and"
- [methods] Compile with max kmer size: "make MAXK=63 metacortex"
- [methods] Run compiled executable: "./bin/metacortex_k63"
- [intro] GCC required for compilation: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [intro] Two-stage execution producing contigs and sequence graphs: "Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
- [methods] Cloning the repository: "git clone https://github.com/SR-Martin/metacortex.git"
