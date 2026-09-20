---
name: contig-n50-benchmark-computation
description: Use when comparing two or more assembly runs (e.g. sparse_pregraph vs
  pregraph builds) and you need a single continuity metric such as contig N50 to quantify
  how a graph-construction choice affects assembly length.
when_to_use_negative:
- When the goal is per-base accuracy or misassembly detection rather than assembly
  continuity — N50 alone rewards longer contigs regardless of correctness (the article
  separately reports structural error, substitution error, and copy number error reductions).
- When scaffold-level continuity is the quantity of interest — use scaffold N50 from
  the *.scafSeq output instead of contig N50 from *.contig.
- When assemblies were built from different read sets or different k-mer settings
  without an explicit tradeoff rationale — N50 comparison is only meaningful for builds
  on identical input.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: SOAPdenovo2
  role: Assembles contigs from the sparse de Bruijn graph build; its *.contig output
    is the input to N50 computation
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo
  role: Predecessor assembler run in dense pregraph mode on identical reads to provide
    the baseline contigs for N50 comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:aa8964eabb8144b6bcea4708d8bc13e2f001128405b83b7a72135e1239c2d5e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - SOAPdenovo
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# contig-n50-benchmark-computation

## Summary

Computes contig N50 from assembled contig sequences to quantify assembly continuity, used here to measure the assembly tradeoff introduced by SOAPdenovo2's sparse de Bruijn graph mode versus the standard dense pregraph build.

## When to use

Use when comparing two or more assembly runs (e.g. sparse_pregraph vs pregraph builds) and you need a single continuity metric such as contig N50 to quantify how a graph-construction choice affects assembly length.

## When NOT to use

- When the goal is per-base accuracy or misassembly detection rather than assembly continuity — N50 alone rewards longer contigs regardless of correctness (the article separately reports structural error, substitution error, and copy number error reductions).
- When scaffold-level continuity is the quantity of interest — use scaffold N50 from the *.scafSeq output instead of contig N50 from *.contig.
- When assemblies were built from different read sets or different k-mer settings without an explicit tradeoff rationale — N50 comparison is only meaningful for builds on identical input.

## Inputs

- *.contig FASTA output from SOAPdenovo2 contig step (contig sequences without using mate pair information)
- Comparable *.contig files from the competing assembler build run on the identical read set
- Peak memory measurements (e.g. /usr/bin/time -v output) captured during graph construction for each mode

## Outputs

- Contig N50 value in base pairs per assembly mode
- Two-row comparison table memory_benchmark.tsv with columns mode, peak_memory_mb, contig_n50_bp

## How to apply

After running contig assembly on the graph build under test (e.g. SOAPdenovo2 'contig -g graph_prefix -R' following a sparse_pregraph or pregraph step), extract the contig sequences from the *.contig output file. Sort all contig lengths in descending order and report the length at which the cumulative sum reaches 50% of the total assembled bases; this is the contig N50. Compute N50 identically for each competing build (sparse-mode SOAPdenovo2 and dense-build predecessor SOAPdenovo) on the same input reads so the values are directly comparable. Record N50 alongside peak resident memory captured with /usr/bin/time -v to expose the memory-versus-continuity tradeoff: the paper reports memory consumption ~2/3 lower at the point of largest memory consumption, and the sparse pregraph module 'usually needs 2-10bp smaller kmer length to achieve the same performance as the original pregraph module', which is the mechanism behind any observed N50 reduction. Write the paired measurements to a table such as memory_benchmark.tsv with columns mode, peak_memory_mb, contig_n50_bp.

## Related tools

- **SOAPdenovo2** (Assembles contigs from the sparse de Bruijn graph build; its *.contig output is the input to N50 computation) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo** (Predecessor assembler run in dense pregraph mode on identical reads to provide the baseline contigs for N50 comparison)

## Examples

```
/usr/bin/time -v ./SOAPdenovo2 sparse_pregraph -s config_file -K 63 -z 5000000000 -R -o graph_prefix; ./SOAPdenovo2 contig -g graph_prefix -R
```

## Evaluation signals

- The N50 computation is invariant to contig ordering in the FASTA: shuffling contig records must not change the reported N50.
- N50 must be ≤ the largest contig length and ≥ half of it by definition of the 50% cumulative-length threshold.
- Sum of contig lengths contributing to the N50 breakpoint equals 50% of total assembled bases (within one contig's length).
- Sparse-mode peak memory (from /usr/bin/time -v) should be lower than the dense pregraph build, consistent with the reported ~2/3 reduction at peak memory consumption; any N50 reduction for the sparse build is recorded as the tradeoff.
- memory_benchmark.tsv parses cleanly with the exact schema: mode, peak_memory_mb, contig_n50_bp — one row per assembly mode.

## Limitations

- N50 rewards long contigs irrespective of correctness; the article supplements it with accuracy metrics (structural error reduced by 90.12%, substitution error by 92.13%, copy number error by 69.47% vs SOAPdenovo1).
- The sparse pregraph module requires an estimated genome size (-z) that 'should be set a litter larger than the real genome size' for memory allocation, and typically needs a 2–10 bp smaller k-mer than the original pregraph, so contig N50 differences may partly reflect the shorter k-mer rather than graph sparsity alone.
- Contig-level N50 excludes mate-pair information; scaffold N50 from *.scafSeq will differ substantially.

## Evidence

- [abstract] Sparse-mode memory reduction is the benchmarked quantity: "memory consumption was ~2/3 lower during the point of largest memory consumption."
- [readme] Sparse pregraph k-mer tradeoff explaining N50 differences: [evidence span withheld — anchor status: external_doc]
- [readme] Contig output file produced by the contig step is the N50 input: [evidence span withheld — anchor status: external_doc]
- [results] N50 is the continuity metric used to benchmark SOAPdenovo2 against competitors: "SOAPdenovo2 produced contig N50 and scaffold N50 that were approximately 1.53 and 1.84-times longer"
