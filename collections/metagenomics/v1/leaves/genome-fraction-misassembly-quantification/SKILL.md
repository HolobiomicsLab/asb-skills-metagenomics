---
name: genome-fraction-misassembly-quantification
description: Use when comparing metagenomic assembler outputs (FASTA contigs from
  tools like MetaCortex, MEGAHIT, metaSPAdes, Ray Meta) and you need per-reference
  genome fraction and misassembly counts to rank assemblers quantitatively.
when_to_use_negative:
- Input is a raw read set rather than assembled contigs — assembly must happen before
  genome fraction/misassembly evaluation
- No reference genomes are available for the community, so genome fraction cannot
  be computed against references
- The goal is diversity capture (sequence graph variants) rather than assembly accuracy
  benchmarking against a consensus reference
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Primary assembler under benchmark; compiled with MAXK=63 and run with minimum
    coverage threshold 5, then its contigs are evaluated for genome fraction and misassemblies
  repo: https://github.com/SR-Martin/metacortex
- name: MEGAHIT
  role: Comparator assembler run with recommended default settings on the identical
    read set
- name: metaSPAdes
  role: Comparator assembler run with recommended default settings on the identical
    read set
- name: Ray Meta
  role: Comparator assembler run with recommended default settings on the identical
    read set
- name: CAMISIM
  role: Generates the simulated viral community reads used as the common input for
    all assemblers
- name: GCC
  role: Compiler required to build MetaCortex from source before benchmarking (LLVM
    will not compile MetaCortex)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:b712aa3224f1e42b905989fd971ec3612458fc278743590cc1ea326cec205167
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex
  tools:
  - CAMISIM
  - GCC
  - MEGAHIT
  - MetaCortex
  - Ray Meta
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genome-fraction-misassembly-quantification

## Summary

Benchmark metagenomic assemblies by computing genome fraction and misassembly counts against reference genomes, as done to show MetaCortex reached 98.78% overall genome fraction with no misassemblies on a CAMISIM-simulated viral community versus MEGAHIT, metaSPAdes and Ray Meta.

## When to use

Use when comparing metagenomic assembler outputs (FASTA contigs from tools like MetaCortex, MEGAHIT, metaSPAdes, Ray Meta) and you need per-reference genome fraction and misassembly counts to rank assemblers quantitatively.

## When NOT to use

- Input is a raw read set rather than assembled contigs — assembly must happen before genome fraction/misassembly evaluation
- No reference genomes are available for the community, so genome fraction cannot be computed against references
- The goal is diversity capture (sequence graph variants) rather than assembly accuracy benchmarking against a consensus reference

## Inputs

- FASTQ paired-end metagenomic reads (2x150 bp, ~15 Gb) from a simulated viral community
- Reference genome FASTA files for all community members (six real virus species plus four simulated Influenza B strains)
- Contig FASTA files produced by each assembler (MetaCortex k=63, MEGAHIT, metaSPAdes, Ray Meta)

## Outputs

- Per-assembler genome fraction statistics (overall and per-reference, e.g. 95.27%–99.91% per genome)
- Per-assembler misassembly counts
- Compiled comparison table (e.g. table5_simulated_viral_assembly_stats.csv)

## How to apply

Assemble the identical read set (here 15 Gb of 2x150 bp paired-end reads from a CAMISIM-simulated community of six real virus species plus four simulated Influenza B strains) with each assembler — MetaCortex compiled with `make MAXK=63 metacortex` and run with a minimum coverage threshold of 5, and comparators with recommended defaults. Evaluate every assembly against the full set of reference genomes, computing genome fraction (percent of each reference covered by contigs) and misassembly counts per assembler. Compile per-assembler statistics into a table (e.g. table5_simulated_viral_assembly_stats.csv) and validate against expected ranges: MetaCortex 98.78% overall genome fraction with no misassemblies, comparators in the 77–93% genome fraction range.

## Related tools

- **MetaCortex** (Primary assembler under benchmark; compiled with MAXK=63 and run with minimum coverage threshold 5, then its contigs are evaluated for genome fraction and misassemblies) — https://github.com/SR-Martin/metacortex
- **MEGAHIT** (Comparator assembler run with recommended default settings on the identical read set)
- **metaSPAdes** (Comparator assembler run with recommended default settings on the identical read set)
- **Ray Meta** (Comparator assembler run with recommended default settings on the identical read set)
- **CAMISIM** (Generates the simulated viral community reads used as the common input for all assemblers)
- **GCC** (Compiler required to build MetaCortex from source before benchmarking (LLVM will not compile MetaCortex))

## Examples

```
make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- MetaCortex overall genome fraction reproduces the reported 98.78% with per-genome fractions in the 95.27%–99.91% range
- MetaCortex assembly reports zero misassemblies against the reference genomes
- Ray Meta and metaSPAdes fall in the 77–93% genome fraction range on the same read set
- All assemblers were run on the identical read set (same CAMISIM simulation), so per-reference statistics are directly comparable
- Compiled per-assembler statistics match the structure of table5_simulated_viral_assembly_stats.csv

## Limitations

- Results are specific to high strain-level diversity viral communities (mock and simulated); rankings may differ on other community types
- Simulated reads from CAMISIM do not capture all real-read artifacts (adapters, quality decay), so trimming steps may be needed for real data
- MetaCortex must be compiled with GCC (LLVM is unsupported) and with MAXK large enough for the chosen k-mer (k=63), or the executable cannot be built/run as benchmarked

## Evidence

- [results] MetaCortex achieved the highest genome fraction on simulated communities: [evidence span withheld — anchor status: not_found]
- [abstract] MetaCortex outperforms comparators on high-diversity communities: [evidence span withheld — anchor status: not_found]
- [methods] Building MetaCortex for the benchmark: "MetaCortex can be installed by compiling the source code with GCC"
- [abstract] Version pinned for reproducibility of reported results: "The version used for the results presented in this article is available at doi.org/10.5281/zenodo.7273627."
