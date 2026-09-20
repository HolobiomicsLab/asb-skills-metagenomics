---
name: metagenomic-assembly-quality-assessment
description: Use when comparing metagenomic assemblers on the same Illumina read set
  (mock or simulated viral/bacterial communities) and you need genome fraction, misassembly,
  and per-100 kb error metrics against reference genomes.
when_to_use_negative:
- No reference genomes are available for the community members, so MetaQUAST cannot
  compute genome fraction, misassemblies, or per-100 kb error rates.
- The goal is de novo assembly quality without comparison — this skill is a benchmarking
  workflow requiring identical read sets assembled by multiple baseline assemblers.
- The data are single-strain isolates without strain-level diversity, the setting
  MetaCortex's local-variation graph search was designed for.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Metagenome assembler under test; builds de Bruijn graphs, outputs CTX files
    and contigs/sequence graphs; compiled with a chosen MAXK or installed via bioconda
  repo: https://github.com/SR-Martin/metacortex.git
- name: MetaQUAST
  role: Evaluates all assemblies against reference genomes to yield genome fraction,
    misassemblies, and mismatches/indels per 100 kb
- name: trim-galore
  role: Adapter/quality-trims reads before assembly for every assembler compared
  repo: https://github.com/FelixKrueger/TrimGalore
- name: SRA toolkit
  role: Downloads raw reads (fastq-dump SRR961514) used as assembly input
  repo: https://github.com/ncbi/sra-tools/wiki
- name: GCC
  role: Compiler required to build MetaCortex from source (LLVM will not compile it)
- name: Conda
  role: Alternative installation channel for MetaCortex (bioconda build supports up
    to kmer 127)
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:d08de1a4356dab1666d76aaa364292566ddd0198584a7742b7770f0e1f390c15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - CAMISIM
  - Conda
  - GCC
  - MEGAHIT
  - MetaCortex
  - MetaQUAST
  - Ray Meta
  - SRA toolkit
  - metaSPAdes
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metagenomic-assembly-quality-assessment

## Summary

Benchmark metagenomic assemblers (MetaCortex vs MEGAHIT, metaSPAdes, MetaVelvet, Ray Meta) on identical read sets and evaluate assemblies with MetaQUAST against reference genomes using genome fraction, misassembly counts, and mismatch/indel rates per 100 kb.

## When to use

Use when comparing metagenomic assemblers on the same Illumina read set (mock or simulated viral/bacterial communities) and you need genome fraction, misassembly, and per-100 kb error metrics against reference genomes.

## When NOT to use

- No reference genomes are available for the community members, so MetaQUAST cannot compute genome fraction, misassemblies, or per-100 kb error rates.
- The goal is de novo assembly quality without comparison — this skill is a benchmarking workflow requiring identical read sets assembled by multiple baseline assemblers.
- The data are single-strain isolates without strain-level diversity, the setting MetaCortex's local-variation graph search was designed for.

## Inputs

- Illumina paired-end FASTQ reads (e.g. SRR961514 Mock A viral community; ERR2099157 human gut reads for in silico HIV mixes)
- Reference genomes of the community members (e.g. 12 Mock A virus genomes, five HIV-1 strains 89.6/HXB2/JR-CSF/NL4-3/YU2, MBARC-26 species) in FASTA
- Adapter-trimmed reads from trim-galore
- CAMISIM-simulated communities (15 Gb viral, 14.9 Gb bacterial, 2x150 bp paired-end)

## Outputs

- CTX binary graph files from parallel per-read-file import
- Assembled contigs (plus sequence graphs / GFA) from combined CTX files
- MetaQUAST evaluation metrics: genome fraction (%), misassembly counts, mismatches and indels per 100 kb
- Collated per-assembler statistics tables (e.g. mock_a_metacortex_k95_quast_stats.tsv, hiv_mix_assembly_stats.tsv, table5_simulated_viral_assembly_stats.csv)

## How to apply

Download reads with fastq-dump (e.g. SRR961514) and adapter-trim them with trim-galore before assembly. Build MetaCortex at a compile-time max k-mer via 'make MAXK=95 metacortex' (choices 31/63/95/127/160/192; Conda build supports up to k=127) or install with 'conda install -c bioconda metacortex', then run the two-stage process: parallel import of trimmed read files into de Bruijn graphs outputting CTX files, followed by combining CTX files into contigs; MetaCortex SW mode uses k=127, delta=0.4, min coverage=25, and bacterial/viral simulations use k=63 with minimum coverage 5. Assemble the identical trimmed reads with MEGAHIT, metaSPAdes, MetaVelvet and Ray Meta using their recommended default parameters. Evaluate all assemblies with MetaQUAST against the reference genomes to obtain genome fraction, misassembly counts, and mismatches/indels normalised per 100 kb. Collate per-assembler metrics into a stats table (e.g. TSV/CSV) and check MetaCortex rows reproduce the reported values.

## Related tools

- **MetaCortex** (Metagenome assembler under test; builds de Bruijn graphs, outputs CTX files and contigs/sequence graphs; compiled with a chosen MAXK or installed via bioconda) — https://github.com/SR-Martin/metacortex.git
- **MetaQUAST** (Evaluates all assemblies against reference genomes to yield genome fraction, misassemblies, and mismatches/indels per 100 kb)
- **trim-galore** (Adapter/quality-trims reads before assembly for every assembler compared) — https://github.com/FelixKrueger/TrimGalore
- **SRA toolkit** (Downloads raw reads (fastq-dump SRR961514) used as assembly input) — https://github.com/ncbi/sra-tools/wiki
- **GCC** (Compiler required to build MetaCortex from source (LLVM will not compile it))
- **Conda** (Alternative installation channel for MetaCortex (bioconda build supports up to kmer 127))

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- Mock A (k=95) row must reproduce: genome fraction 99.89%, zero misassemblies, 4.43 mismatches and 3.44 indels per 100 kb.
- Simulated viral community: MetaCortex genome fraction must reach 98.78% with no misassemblies, while Ray Meta/metaSPAdes fall in the 77–93% range.
- Simulated bacterial community: MetaCortex (k=63, min coverage=5) genome fraction ≈71%, with baselines in the reported 27–54% range.
- HIV lab mix: MetaCortex SW (k=127, delta=0.4, min coverage=25) yields 82.563% genome fraction across 31 contigs with no misassemblies.
- All assemblers must be run on the identical trimmed read set and evaluated by the same MetaQUAST configuration against the same reference genomes.

## Limitations

- Conda build of MetaCortex supports k-mer sizes only up to 127, so k=95 and k=127 configurations are covered but higher MAXK values (160, 192) require source compilation.
- LLVM will not compile MetaCortex; Mac users must install GCC and set CC to point to the GCC binary.
- Validation is exacting: reported statistics (e.g. 99.89%, 4.43 mismatches/100 kb) must be reproduced from the specified read accessions and simulation parameters (CAMISIM 15 Gb / 14.9 Gb 2x150 bp runs).

## Evidence

- [abstract] MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers: "MetaCortex produces accurate assemblies with higher genome coverage and contiguity than other popular metagenomic assemblers"
- [intro] Two-stage assembly with CTX files: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [intro] Trimming before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [methods] Conda installation: "conda install -c bioconda metacortex"
- [readme] Trim Galore purpose: [evidence span withheld — anchor status: external_doc]
