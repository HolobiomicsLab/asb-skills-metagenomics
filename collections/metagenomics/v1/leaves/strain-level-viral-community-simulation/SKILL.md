---
name: strain-level-viral-community-simulation
description: Use when benchmarking metagenomic assemblers on communities with high
  strain-level diversity, where a simulated ground-truth community with known reference
  genomes is needed to compute per-genome fraction and misassembly counts.
when_to_use_negative:
- Input is real environmental metagenomic data without reference genomes — genome
  fraction and misassembly validation against ground truth is not possible
- The research question concerns isolate (single-strain) assembly rather than intra-species
  strain-level variation within a mixed community
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: CAMISIM
  role: Simulates the viral metagenomic community and generates the paired-end reads
    used as benchmark input
- name: MetaCortex
  role: Assembles the simulated reads with k=63 and minimum coverage threshold 5;
    the assembler under evaluation
  repo: https://github.com/SR-Martin/metacortex
- name: MEGAHIT
  role: Comparison assembler run on the identical read set with recommended default
    settings
- name: metaSPAdes
  role: Comparison assembler run on the identical read set with recommended default
    settings
- name: Ray Meta
  role: Comparison assembler run on the identical read set with recommended default
    settings
- name: GCC
  role: Compiler required to build MetaCortex from source with make MAXK=63
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:c13be0238af984ce4b770869e6c1279b8ef3ad2be11706e99ea6cb62da804c61
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
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# strain-level-viral-community-simulation

## Summary

Generates a CAMISIM-simulated viral community containing real virus species plus simulated Influenza B strains to benchmark metagenomic assemblers on strain-level variation, evaluated by genome fraction and misassemblies (e.g. MetaCortex achieving 98.78% genome fraction with no misassemblies).

## When to use

Use when benchmarking metagenomic assemblers on communities with high strain-level diversity, where a simulated ground-truth community with known reference genomes is needed to compute per-genome fraction and misassembly counts.

## When NOT to use

- Input is real environmental metagenomic data without reference genomes — genome fraction and misassembly validation against ground truth is not possible
- The research question concerns isolate (single-strain) assembly rather than intra-species strain-level variation within a mixed community

## Inputs

- Reference genomes for six real virus species
- Reference genomes for four simulated Influenza B strains
- CAMISIM simulation parameters for 15 Gb of 2x150 bp paired-end reads
- Compiled MetaCortex executable (metacortex_k63) and default-setting installations of MEGAHIT, metaSPAdes, and Ray Meta

## Outputs

- Simulated 2x150 bp paired-end FASTQ reads (~15 Gb) from the CAMISIM viral community
- Contig assemblies from each assembler (MetaCortex k=63, MEGAHIT, metaSPAdes, Ray Meta)
- table5_simulated_viral_assembly_stats.csv with per-assembler genome fraction and misassembly counts

## How to apply

Regenerate the CAMISIM-simulated viral community from the reference set of six real virus species plus four simulated Influenza B strains, producing 15 Gb of 2x150 bp paired-end reads per the Methods simulation parameters. Compile MetaCortex from source with a k-mer setting appropriate to the reads (make MAXK=63 metacortex) and run the metacortex_k63 executable with a minimum coverage threshold of 5. Assemble the identical read set with comparison assemblers (MEGAHIT, metaSPAdes, Ray Meta) using their recommended default settings so differences reflect assembler behaviour rather than tuning. Evaluate every assembly against the reference genomes, computing genome fraction and misassembly counts per assembler, then compile per-assembler statistics into a CSV such as table5_simulated_viral_assembly_stats.csv. Rationale: simulated strains with known references make it possible to measure exactly how much of each genome an assembler recovers and whether it introduces errors, which is impossible with purely environmental data.

## Related tools

- **CAMISIM** (Simulates the viral metagenomic community and generates the paired-end reads used as benchmark input)
- **MetaCortex** (Assembles the simulated reads with k=63 and minimum coverage threshold 5; the assembler under evaluation) — https://github.com/SR-Martin/metacortex
- **MEGAHIT** (Comparison assembler run on the identical read set with recommended default settings)
- **metaSPAdes** (Comparison assembler run on the identical read set with recommended default settings)
- **Ray Meta** (Comparison assembler run on the identical read set with recommended default settings)
- **GCC** (Compiler required to build MetaCortex from source with make MAXK=63)

## Examples

```
make MAXK=63 metacortex && ./bin/metacortex_k63 --input simulated_reads.fq --out Assembly --min_cov 5
```

## Evaluation signals

- MetaCortex achieves 98.78% overall genome fraction on the simulated viral community, with per-genome fractions between 95.27% and 99.91%
- MetaCortex produces no misassemblies on the simulated community
- Ray Meta and metaSPAdes fall in the 77–93% genome fraction range on the same read set
- All assemblers were run on the identical read set (MetaCortex the only one with non-default k=63 / min coverage 5 settings), so output stats are directly comparable
- Compiled stats table matches the reported Table 5 values for genome fraction and misassembly counts

## Limitations

- Results depend on the specific CAMISIM simulation parameters (15 Gb, 2x150 bp reads); other read lengths or coverages may shift assembler rankings
- Benchmark validity rests on simulated strains; real communities with unknown strain diversity may behave differently
- LLVM will not compile MetaCortex, so builds must use GCC (Mac users must set CC to point to the GCC binary)

## Evidence

- [results] CAMISIM-simulated viral community benchmark: [evidence span withheld — anchor status: not_found]
- [methods] simulation workflow: [evidence span withheld — anchor status: not_found]
- [methods] build and run parameters: [evidence span withheld — anchor status: not_found]
- [abstract] capability claim: [evidence span withheld — anchor status: not_found]
- [intro] compiler requirement: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
