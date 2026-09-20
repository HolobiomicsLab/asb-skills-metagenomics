---
name: viral-metagenome-assembly
description: Use when assembling low-abundance viral genomes (e.g. mock viral communities
  or strain mixes spiked into host reads) from FASTQ reads and you need strain-level
  genome fraction, misassembly, and mismatch/indel metrics per assembler.
when_to_use_negative:
- Input is already assembled contigs or a reference-mapped analysis — MetaCortex is
  a de novo metagenome assembler for environmental sequence reads.
- MacOS user relying on LLVM as the compiler — LLVM will not compile MetaCortex; GCC
  with CC pointing at the GCC binary is required.
- K-mer sizes above 127 are needed and installation is via Conda — the Conda build
  supports only up to kmer 127, so source compilation with make MAXK=160/192 is required
  instead.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3174
tools:
- name: MetaCortex
  role: Two-stage de Bruijn graph metagenomic assembler (CTX import then CTX combination)
    that captures intra-species diversity; run with MAXK/k-mer choices 31/63/95/127/160/192
    or SW mode k=127
  repo: https://github.com/SR-Martin/metacortex
- name: SRA toolkit
  role: Downloads the viral community reads with fastq-dump (e.g. SRR961514)
  repo: https://github.com/ncbi/sra-tools
- name: trim-galore
  role: Adapter-trims reads before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
- name: GCC
  role: Compiles MetaCortex from source (LLVM will not compile it); make MAXK=<value>
- name: Conda
  role: Installs MetaCortex via bioconda (kmer up to 127)
- name: MetaQUAST
  role: Evaluates all assemblies against reference virus genomes for genome fraction,
    misassemblies, and mismatch/indel rates per 100 kb
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:5321c469fa8f49198228f182e935948908886127375cff01559b276def413389
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex
  tools:
  - Conda
  - GCC
  - MetaCortex
  - MetaQUAST
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# viral-metagenome-assembly

## Summary

Assemble viral genomes from metagenomic Illumina reads with MetaCortex, a Cortex-derived de Bruijn graph assembler that captures intra-species diversity from the assembly graph, and benchmark the contigs against reference viral genomes with MetaQUAST.

## When to use

Use when assembling low-abundance viral genomes (e.g. mock viral communities or strain mixes spiked into host reads) from FASTQ reads and you need strain-level genome fraction, misassembly, and mismatch/indel metrics per assembler.

## When NOT to use

- Input is already assembled contigs or a reference-mapped analysis — MetaCortex is a de novo metagenome assembler for environmental sequence reads.
- MacOS user relying on LLVM as the compiler — LLVM will not compile MetaCortex; GCC with CC pointing at the GCC binary is required.
- K-mer sizes above 127 are needed and installation is via Conda — the Conda build supports only up to kmer 127, so source compilation with make MAXK=160/192 is required instead.

## Inputs

- Illumina metagenomic FASTQ reads (e.g. SRR961514 mock viral community from SRA; ERR2099157 human gut reads from ENA)
- In-silico mixed read sets (e.g. five HIV-1 strains 89.6, HXB2, JR-CSF, NL4-3, YU-2 spiked into human gut reads at ~2.8% total abundance)
- Trimmed FASTQ reads (post trim-galore)
- Reference viral genome FASTA files for MetaQUAST evaluation

## Outputs

- MetaCortex contigs (and sequence graphs in GFA format) from the combined CTX files
- MetaQUAST statistics per assembler: genome fraction, misassembly counts, mismatches and indels per 100 kb
- Per-benchmark result tables, e.g. mock_a_metacortex_k95_quast_stats.tsv and hiv_mix_assembly_stats.tsv

## How to apply

Obtain reads with fastq-dump from the SRA toolkit (e.g. SRR961514) and adapter-trim them with trim-galore before assembly. Install MetaCortex either via 'conda install -c bioconda metacortex' (kmer up to 127) or by compiling from source with a chosen maximum k-mer (make MAXK=31/63/95/127/160/192), using GCC since LLVM will not compile MetaCortex. Run the two-stage assembly: first import each trimmed reads file into a de Bruijn graph and output a CTX file in parallel, then combine the separate CTX files to output contigs (and sequence graphs); for high strain-diversity mixes use SW mode, e.g. k=127, delta=0.4, min coverage=25 for the in-silico HIV mix at ~2.8% total abundance, and k=95 for the Mock A 12-virus community. Assemble the same trimmed reads with baseline metagenomic assemblers (MEGAHIT, metaSPAdes, Ray Meta) for comparison. Evaluate all assemblies with MetaQUAST against the reference virus genomes to obtain genome fraction, misassembly counts, and mismatches/indels normalised per 100 kb, then tabulate per-assembler metrics into a TSV.

## Related tools

- **MetaCortex** (Two-stage de Bruijn graph metagenomic assembler (CTX import then CTX combination) that captures intra-species diversity; run with MAXK/k-mer choices 31/63/95/127/160/192 or SW mode k=127) — https://github.com/SR-Martin/metacortex
- **SRA toolkit** (Downloads the viral community reads with fastq-dump (e.g. SRR961514)) — https://github.com/ncbi/sra-tools
- **trim-galore** (Adapter-trims reads before assembly) — https://github.com/FelixKrueger/TrimGalore
- **GCC** (Compiles MetaCortex from source (LLVM will not compile it); make MAXK=<value>)
- **Conda** (Installs MetaCortex via bioconda (kmer up to 127))
- **MetaQUAST** (Evaluates all assemblies against reference virus genomes for genome fraction, misassemblies, and mismatch/indel rates per 100 kb)

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=127 metacortex
```

## Evaluation signals

- Mock A (k=95) row reproduces reported stats: genome fraction 99.89%, zero misassemblies, 4.43 mismatches per 100 kb, and 3.44 indels per 100 kb.
- HIV lab mix (SW mode, k=127, delta=0.4, min coverage=25) reproduces 82.563% genome fraction across 31 contigs with no misassemblies.
- MetaCortex achieves higher genome coverage and contiguity than baseline assemblers (MEGAHIT, metaSPAdes, Ray Meta) on high strain-diversity viral communities.
- Both assembly stages complete: parallel CTX files are produced per reads file and are successfully combined into contigs/sequence graphs.
- All evaluated assemblies are compared on the same trim-galore-trimmed input reads with identical MetaQUAST references.

## Limitations

- LLVM will not compile MetaCortex, so Mac users must install GCC and set CC to point at the GCC binary.
- The Conda build supports k-mer sizes only up to 127; larger MAXK values (160, 192) require building from source.
- Reported benchmark gains are for mock viral communities with high strain-level diversity and simulated communities; performance on other data types is not established here.

## Evidence

- [abstract] MetaCortex captures intra-species diversity by searching for signatures of local variation along assembled sequences in the assembly graph and outputs sequences in sequence graph format: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [intro] Two-stage assembly process: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [methods] Conda install method: "conda install -c bioconda metacortex"
- [intro] Trimming requirement: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [intro] GCC requirement on Mac: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [results] Mock A benchmark result: [evidence span withheld — anchor status: not_found]
- [results] HIV mix benchmark result: [evidence span withheld — anchor status: not_found]
- [readme] Clone source from GitHub: "git clone https://github.com/SR-Martin/metacortex.git"
