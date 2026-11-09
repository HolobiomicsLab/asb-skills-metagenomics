---
name: bwt-construction-runtime-profiling
description: Use when benchmarking BWT-based read-analysis pipelines on raw FASTA/FASTQ
  read sets and you must report wall-clock split between index construction (BCR/eBWT)
  and downstream SNP/INDEL detection.
when_to_use_negative:
- The goal is INDEL detection with the parallel pipeline — the parallel version works
  for SNPs only because HARC pre-processing does not account for INDELs in the read
  set
- Input is already an indexed or aligned dataset (BAM/CRAM or a precomputed BWT),
  since the skill profiles construction time from raw reads only
- Timing the sequential variant-detection step without excluding eBWT computation,
  which the paper explicitly excludes from the reported computational cost of EBWT2INDEL
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: BCR_LCP_GSA
  role: Builds the multi-string eBWT of the read collection; its runtime is recorded
    as a separate line item from variant detection
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: ebwt2InDel
  role: Performs the SNP/INDEL detection whose wall-clock time is profiled, in both
    single-sample sequential mode and parallel pieces
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: HARC
  role: External pre-processing tool that sorts the fasta by context before splitting
    into 24 pieces for the parallel pipeline
  repo: https://github.com/shubhamchandak94/HARC
- name: SimSeq
  role: Simulates the synthetic Human chromosome 1 reads at 30x coverage with the
    HiSeq error profile for reproducible benchmarking
- name: DISCOSNP++
  role: State-of-the-art competitor whose multi-threaded runtime is tabulated for
    comparison with the parallel pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:c919ac70b725920e0dc0a1ee693690c0ae1bafa33e070ca8b9b67c4232b14be8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/giovannarosone/BCR_LCP_GSA
  tools:
  - BCR_LCP_GSA
  - DISCOSNP++
  - HARC
  - SimSeq
  - ebwt2InDel
  - pebwt2InDel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# bwt-construction-runtime-profiling

## Summary

A benchmarking skill for reference-free variant discovery pipelines that profiles BWT construction time separately from variant-detection time, and compares sequential versus parallel (multi-core) runtimes against a state-of-the-art competitor such as DISCOSNP++.

## When to use

Use when benchmarking BWT-based read-analysis pipelines on raw FASTA/FASTQ read sets and you must report wall-clock split between index construction (BCR/eBWT) and downstream SNP/INDEL detection.

## When NOT to use

- The goal is INDEL detection with the parallel pipeline — the parallel version works for SNPs only because HARC pre-processing does not account for INDELs in the read set
- Input is already an indexed or aligned dataset (BAM/CRAM or a precomputed BWT), since the skill profiles construction time from raw reads only
- Timing the sequential variant-detection step without excluding eBWT computation, which the paper explicitly excludes from the reported computational cost of EBWT2INDEL

## Inputs

- Raw read dataset in FASTA/FASTQ format (single-sample or two-sample mode), with reads filtered to contain only A/C/G/T (reads containing N removed or replaced)
- Reverse-complement copies of reads added to the dataset to improve sensitivity
- Machine specification (e.g. 24-core server) and per-tool RAM budgets in MB

## Outputs

- Tabulated wall-clock times separating BWT construction (BCR) from ebwt2InDel SNP/INDEL detection for sequential and parallel runs
- Parallel-versus-sequential speedup ratio (expected about one order of magnitude, ~10x; reference point: 33 minutes on 24 cores)
- Comparison table against the state-of-the-art competitor's runtime under multi-threading

## How to apply

Generate or obtain a raw read dataset (e.g. synthetic Human chromosome 1 reads simulated at 30x coverage with SimSeq using the HiSeq error profile, coverage uniformly distributed between the two chromosome variants). Run the sequential pipeline: build the eBWT with BCR_LCP_GSA using an explicit RAM budget in MB (e.g. 1024), then run ebwt2InDel in single-sample mode, recording BWT construction time and SNP-detection wall-clock time as separate line items. Then run the parallel pipeline via the pebwt2InDel script, which sorts the fasta by context with HARC, splits it into 24 pieces, builds per-piece BWTs with BCR_LCP_GSA, and runs ebwt2InDel in parallel on 24 cores. Tabulate sequential and parallel runtimes side by side with the state-of-the-art tool's multi-threaded runtime. Report construction and detection times separately, since the paper explicitly excludes eBWT computation from the reported computational cost of the detection algorithm.

## Related tools

- **BCR_LCP_GSA** (Builds the multi-string eBWT of the read collection; its runtime is recorded as a separate line item from variant detection) — https://github.com/giovannarosone/BCR_LCP_GSA
- **ebwt2InDel** (Performs the SNP/INDEL detection whose wall-clock time is profiled, in both single-sample sequential mode and parallel pieces) — https://github.com/nicolaprezza/ebwt2InDel
- **HARC** (External pre-processing tool that sorts the fasta by context before splitting into 24 pieces for the parallel pipeline) — https://github.com/shubhamchandak94/HARC
- **SimSeq** (Simulates the synthetic Human chromosome 1 reads at 30x coverage with the HiSeq error profile for reproducible benchmarking)
- **DISCOSNP++** (State-of-the-art competitor whose multi-threaded runtime is tabulated for comparison with the parallel pipeline)

## Examples

```
./eGap --lcp -m 4096 dataset/reads.fastq
```

## Evaluation signals

- Parallel pipeline runtime is within one order of magnitude (about 10x faster) of the sequential pipeline on the same 24-core machine
- The parallel pipeline's total wall-clock time is comparable to DISCOSNP++ run with multi-threading, e.g. approximately 33 minutes on the 30x Human chromosome 1 dataset
- BWT construction time (BCR) and EBWT2INDEL detection time are reported as separate values, never merged into a single number
- Sequential run uses an explicit RAM budget for BCR_LCP_GSA (e.g. 1024 MB) and the parallel run uses the same number of pieces as cores (24)
- Speedup is reproduced on the same dataset with coverage uniformly distributed across the two chromosome variants

## Limitations

- The parallel pipeline at the moment works for SNPs only; INDEL support is deferred to a forthcoming update
- HARC pre-processing does not take into account the presence of INDELs in the read set
- Absolute runtimes are machine-dependent; the article's reference points (23h19m BCR build, 33-minute parallel pipeline) apply to a 24-core machine and must be re-measured on different hardware
- No software version numbers are stated for EBWT2INDEL or the competitor, so exact version reproduction is not possible from the paper alone

## Evidence

- [results] We separate the running time required to build the BWT (by using BCR [42, 43]) from the running time of EBWT2INDEL: "We separate the running time required to build the BWT (by using BCR [42, 43]) from the running time of EBWT2INDEL"
- [results] Parallel pipeline terminated in 33 minutes: "The parallel pipeline using HARC followed by BCR and EBWT2INDEL (run in parallel on 24 pieces of the sorted fasta file) terminated in 33 minutes."
- [abstract] Parallel version one order of magnitude faster: "On a 24-cores machine, the parallel version of our tool is one order of magnitude faster than the sequential one"
- [discussion] As a limitation, our parallel pipeline at the moment works for SNPs only: "As a limitation, our parallel pipeline at the moment works for SNPs only"
- [methods] First, the BWT of the input dataset(s) must be built.: "First, the BWT of the input dataset(s) must be built."
