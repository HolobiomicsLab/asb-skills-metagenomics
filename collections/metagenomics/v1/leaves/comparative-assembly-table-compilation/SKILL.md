---
name: comparative-assembly-table-compilation
description: Use when several de novo assemblers (e.g. PEHaplo, SPAdes, SAVAGE, SGA)
  have been run on the same input reads and their contigs must be benchmarked against
  known reference haplotypes with MetaQUAST.
when_to_use_negative:
- The contigs come from different, non-identical read sets — cross-assembler comparison
  on unequal inputs invalidates the benchmark
- The goal is abundance or variant calling rather than strain-level assembly quality
  benchmarking
- No reference haplotypes are available, so MetaQUAST coverage/completeness metrics
  cannot be computed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: TAR-VIR
  role: Pipeline whose overlap-extension read recruiting stage produces the shared
    recruited read set used as the identical benchmarking input for all assemblers
  repo: https://github.com/chjiao/TAR-VIR
- name: PEHaplo
  role: TAR-VIR's default haplotype reconstruction assembler, invoked with its '-f'
    option for a single fasta of mixed single-/paired-end reads; the candidate being
    benchmarked
  repo: github.com/chjiao/PEHaplo
- name: SPAdes
  role: Generic de novo baseline assembler run on the identical recruited reads for
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:3edc81be7073a427786117987605324e28bbe4435b57db5fb3c6dfe00b9be4c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - PEHaplo
  - SPAdes
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# comparative-assembly-table-compilation

## Summary

Compiles MetaQUAST evaluation metrics (N50, genome coverage, contig count, mismatch rate) from multiple de novo assemblers run on the identical recruited read set into a single comparative TSV table, so assembler performance can be judged side-by-side.

## When to use

Use when several de novo assemblers (e.g. PEHaplo, SPAdes, SAVAGE, SGA) have been run on the same input reads and their contigs must be benchmarked against known reference haplotypes with MetaQUAST.

## When NOT to use

- The contigs come from different, non-identical read sets — cross-assembler comparison on unequal inputs invalidates the benchmark
- The goal is abundance or variant calling rather than strain-level assembly quality benchmarking
- No reference haplotypes are available, so MetaQUAST coverage/completeness metrics cannot be computed

## Inputs

- recruited_reads.fa — the recruited viral read set in FASTA (mixed single- and paired-end reads)
- Contig sets (FASTA) from PEHaplo, SPAdes, SAVAGE, and SGA run on the same read set
- Reference haplotype genomes (e.g. five SARS-CoV haplotypes derived from NC_004718.3)

## Outputs

- sars_cov_assembly_comparison.tsv — per-assembler MetaQUAST metrics table (contig count, N50, genome coverage, mismatch rate)
- MetaQUAST per-assembler reports of haplotype coverage and assembly completeness

## How to apply

Run each candidate assembler (TAR-VIR's default PEHaplo via its '-f' option for mixed single-/paired-end reads, plus SPAdes, SAVAGE, and SGA) on the identical recruited read set, e.g. recruited_reads.fa produced by the overlap-extension step. Evaluate all resulting contig sets against the target haplotypes (five SARS-CoV haplotypes derived from NC_004718.3) using MetaQUAST, recording haplotype coverage and assembly completeness per assembler. Compile the per-assembler metrics into a comparative table (sars_cov_assembly_comparison.tsv) so columns can be compared at a fixed read-mapping seed setting such as Bowtie2 'L,0,-0.9'. Judge the winner by consistent superiority in N50 and genome coverage with a low mismatch rate, not by read count alone (BWA aligned slightly more reads than Bowtie2, yet the Bowtie2-recruited reads covered 20%-30% more genome).

## Related tools

- **TAR-VIR** (Pipeline whose overlap-extension read recruiting stage produces the shared recruited read set used as the identical benchmarking input for all assemblers) — https://github.com/chjiao/TAR-VIR
- **PEHaplo** (TAR-VIR's default haplotype reconstruction assembler, invoked with its '-f' option for a single fasta of mixed single-/paired-end reads; the candidate being benchmarked) — github.com/chjiao/PEHaplo
- **SPAdes** (Generic de novo baseline assembler run on the identical recruited reads for comparison)

## Examples

```
./overlap -S align.sam -x prefix -f reads.fa -c 180 -o recruited_reads.fa; then assemble with PEHaplo '-f' and baselines, and compile MetaQUAST metrics into sars_cov_assembly_comparison.tsv
```

## Evaluation signals

- All contig sets were generated from byte-identical recruited read input (same recruited_reads.fa)
- MetaQUAST evaluation is anchored to the correct haplotype references (five SARS-CoV haplotypes from NC_004718.3)
- Per-assembler metrics (contigs, N50, genome coverage %, mismatch %) are reported under one fixed mapping seed set, e.g. Bowtie2 'L,0,-0.9'
- Reference-reported success profile: 7 contigs, N50 9,676, 98.9% genome coverage, 0.0% mismatch rate for PEHaplo under 'L,0,-0.9'
- Ranking is judged on coverage/N50 consistency across tools, matching the paper's claim that TAR-VIR 'competes favorably with other tested tools'

## Limitations

- Read recruitment counts alone are misleading: BWA aligned 52,688 reads vs 52,377 for Bowtie2 'L,0,-0.9', but the Bowtie2-recruited reads covered 20%-30% more genomes — coverage, not read count, should drive conclusions
- MetaQUAST benchmarking requires known reference haplotypes; it cannot score assemblies of completely novel viruses
- Evaluation is specific to the simulated dataset and seed setting; metrics may differ under other error functions such as the recommended 'L,0,-0.6'

## Evidence

- [results] On the recruited SARS-CoV reads, PEHaplo (TAR-VIR's default assembler) consistently achieved higher N50 and genome coverage than the other tested assembly tools: [evidence span withheld — anchor status: not_found]
- [results] BWA reads vs coverage tradeoff: [evidence span withheld — anchor status: not_found]
- [readme] README usage of PEHaplo -f for mixed reads: "The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file."
- [readme] README overlap-extension recruiting command: "./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa"
- [abstract] Abstract favorable-comparison claim: "The results demonstrated that TAR-VIR competes favorably with other tested tools."
