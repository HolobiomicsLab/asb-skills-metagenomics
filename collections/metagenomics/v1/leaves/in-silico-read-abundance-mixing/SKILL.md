---
name: in-silico-read-abundance-mixing
description: 'Use when benchmarking metagenomic assembly of strain-diverse viral communities
  requires a controlled ground truth: target-organism reads must be spiked into a
  complex background at a known abundance before assembly.'
when_to_use_negative:
- The target community reads are already part of the natural background sample, so
  no artificial spike-in is needed and abundance is not experimentally controlled
- The goal is de novo assembly of a single isolate rather than a benchmark of metagenomic
  assembly at controlled strain abundance
- An existing ground-truth benchmark dataset with known per-strain abundances is already
  available
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0797
tools:
- name: SRA toolkit
  role: Download the mock viral community reads (SRR961514) with the fastq-dump command
    before mixing
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Trim the mixed reads with trim-galore before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
- name: MetaCortex
  role: Assemble the mixed reads in SW mode (k=127, delta=0.4, min coverage=25) after
    parallel CTX import and combination into contigs
  repo: https://github.com/SR-Martin/metacortex
- name: MetaQUAST
  role: Evaluate assemblies of the mixed community against the HIV-1 references to
    quantify genome fraction and misassemblies
- name: GCC
  role: Compile MetaCortex from source with a chosen MAXK value (LLVM will not compile
    MetaCortex)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:7f5f3d07f1268383f00ee1a6c09a44068f5a08c1314a915d5b3a4cbf73b3df1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCC
  - MetaCortex
  - MetaQUAST
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# in-silico read abundance mixing

## Summary

Construct a synthetic low-abundance mock community by mixing sequencing reads from target organisms (e.g. five HIV-1 strains) into a background metagenomic read set (human gut) at a defined total abundance (~2.8%), enabling controlled benchmarking of metagenomic assemblers like MetaCortex.

## When to use

Use when benchmarking metagenomic assembly of strain-diverse viral communities requires a controlled ground truth: target-organism reads must be spiked into a complex background at a known abundance before assembly.

## When NOT to use

- The target community reads are already part of the natural background sample, so no artificial spike-in is needed and abundance is not experimentally controlled
- The goal is de novo assembly of a single isolate rather than a benchmark of metagenomic assembly at controlled strain abundance
- An existing ground-truth benchmark dataset with known per-strain abundances is already available

## Inputs

- Mock viral community reads (SRR961514, via fastq-dump)
- Human gut metagenomic reads (ERR2099157, from ENA) as background
- Reference sequences for the five HIV-1 strains (89.6, HXB2, JR-CSF, NL4-3, YU2)

## Outputs

- Mixed FASTQ read set with target strains at ~2.8% total abundance
- Trimmed mixed reads (trim-galore output)
- MetaCortex assembly: contigs and sequence graphs from combined CTX files
- MetaQUAST evaluation table (genome fraction, contig count, misassembly counts), e.g. hiv_mix_assembly_stats.tsv

## How to apply

Download the mock viral community reads (SRR961514) with fastq-dump from the SRA toolkit and the human gut reads (ERR2099157) from ENA. Mix reads from the target strains (HIV-1 89.6, HXB2, JR-CSF, NL4-3, YU2) in silico into the human gut read set so the targets total ~2.8% abundance. Trim the mixed reads with trim-galore before assembly. Assemble with MetaCortex built with make MAXK=127 and run in SW mode with k=127, delta=0.4, min coverage=25 (parallel CTX import followed by combining CTX files into contigs). Evaluate the resulting assemblies against the HIV-1 references with MetaQUAST, tabulating genome fraction, contig count and misassembly counts.

## Related tools

- **SRA toolkit** (Download the mock viral community reads (SRR961514) with the fastq-dump command before mixing) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Trim the mixed reads with trim-galore before assembly) — https://github.com/FelixKrueger/TrimGalore
- **MetaCortex** (Assemble the mixed reads in SW mode (k=127, delta=0.4, min coverage=25) after parallel CTX import and combination into contigs) — https://github.com/SR-Martin/metacortex
- **MetaQUAST** (Evaluate assemblies of the mixed community against the HIV-1 references to quantify genome fraction and misassemblies)
- **GCC** (Compile MetaCortex from source with a chosen MAXK value (LLVM will not compile MetaCortex))

## Examples

```
fastq-dump SRR961514; trim_galore --paired mixed_reads_R1.fastq.gz mixed_reads_R2.fastq.gz
```

## Evaluation signals

- MetaQUAST reports ~82.563% genome fraction across the five HIV-1 genomes with 31 contigs and no misassemblies for MetaCortex SW (k=127, delta=0.4, min coverage=25)
- The spike-in reads constitute ~2.8% of the total mixed read set
- All mixed reads pass through trim-galore cleanly before assembly
- The comparison table hiv_mix_assembly_stats.tsv contains genome fraction, contig count and misassembly counts for every assembler run on the same mixed read set
- Assembly is run in the documented two-stage pattern: parallel import to CTX files, then combination into contigs and sequence graphs

## Limitations

- LLVM will not compile MetaCortex; Mac users must install GCC and set CC to point to the GCC binary
- Only one mix ratio (~2.8% total abundance) and one background (human gut, ERR2099157) were tested in this benchmark; other ratios are unvalidated
- Results depend on accurate reference sequences for each strain when evaluating with MetaQUAST

## Evidence

- [results] MetaCortex SW (k=127, delta=0.4, min coverage=25) assembled 82.563% of the five HIV genomes across 31 contigs with no misassemblies on the in silico mixed HIV lab-mix dataset.: [evidence span withheld — anchor status: not_found]
- [methods] Mix the HIV-1 strain reads at ~2.8% total abundance into the human gut read set before trimming and assembly.: [evidence span withheld — anchor status: not_found]
- [intro] Download the reads using the fastq-dump command from the SRA toolkit: "Download the reads using the fastq-dump command from the SRA toolkit::

	fastq-dump SRR961514"
- [intro] we will use trim-galore to trim the reads before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
- [intro] MetaCortex is executed in two stages: parallel import of reads into CTX files, then combining CTX files to output contigs and sequence graphs.: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
