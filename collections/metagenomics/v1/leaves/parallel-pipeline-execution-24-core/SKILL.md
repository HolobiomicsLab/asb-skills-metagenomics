---
name: parallel-pipeline-execution-24-core
description: Use when a reference-free SNP-discovery job on a large read set (e.g.
  30x Human chromosome 1) is too slow sequentially and a multi-core machine (24 cores)
  is available.
when_to_use_negative:
- INDEL discovery is required — the parallel pipeline works for SNPs only, and the
  external pre-processing tool does not account for INDELs in the read set
- The input reads have not been pre-filtered to contain only 'A','C','G','T' (reads
  containing 'N' must be removed or replaced before running)
- Only a single-core machine is available, so no parallel speedup over the sequential
  pipeline can be realized
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HARC
  role: Pre-processing tool that sorts the fasta by context before splitting into
    24 pieces
  repo: https://github.com/shubhamchandak94/HARC
- name: BCR_LCP_GSA
  role: Builds the per-piece BWT for each of the 24 sorted fasta pieces (RAM budget
    in MB, e.g. 1024)
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: ebwt2InDel
  role: Runs in parallel on the 24 BWT pieces to detect SNPs
  repo: github.com/nicolaprezza/ebwt2InDel
- name: pebwt2InDel
  role: Driver script that orchestrates HARC sorting, piece splitting, per-piece BWT
    construction, and parallel ebwt2InDel execution on 24 cores
  repo: github.com/nicolaprezza/ebwt2InDel
- name: DISCOSNP++
  role: State-of-the-art competitor tool run with multi-threading as the runtime baseline
    for comparison
- name: SimSeq
  role: Generates the synthetic 30x-covered Human chromosome 1 read dataset used to
    benchmark sequential vs parallel runtimes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:dd7a1788ad7f0f1d99319500000f54990bb9a93091ae71c8c66decc0438504bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shubhamchandak94/HARC
  tools:
  - BCR_LCP_GSA
  - DISCOSNP++
  - HARC
  - SimSeq
  - ebwt2InDel
  - pebwt2InDel
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# parallel-pipeline-execution-24-core

## Summary

Runs the reference-free SNP-discovery pipeline in parallel by pre-sorting the fasta with HARC, splitting it into 24 pieces, building per-piece BWTs with BCR_LCP_GSA, and running ebwt2InDel concurrently via pebwt2InDel, achieving roughly 10x speedup over the sequential pipeline.

## When to use

Use when a reference-free SNP-discovery job on a large read set (e.g. 30x Human chromosome 1) is too slow sequentially and a multi-core machine (24 cores) is available.

## When NOT to use

- INDEL discovery is required — the parallel pipeline works for SNPs only, and the external pre-processing tool does not account for INDELs in the read set
- The input reads have not been pre-filtered to contain only 'A','C','G','T' (reads containing 'N' must be removed or replaced before running)
- Only a single-core machine is available, so no parallel speedup over the sequential pipeline can be realized

## Inputs

- FASTA/FASTQ read collection (e.g. 30x-covered synthetic Human chromosome 1 reads from SimSeq with HiSeq error profile)
- 24-way split of the HARC-sorted fasta file

## Outputs

- .snp file with detected SNPs in KisSNP2-format fasta output from each parallel ebwt2InDel run
- Wall-clock runtimes for parallel pipeline (target: ~33 minutes) and separately recorded BWT construction times

## How to apply

First sort the input fasta by context using HARC, then split the sorted fasta into 24 pieces and build a per-piece BWT with BCR_LCP_GSA (with an explicit RAM budget in MB, e.g. 1024). Launch the pebwt2InDel script, which runs ebwt2InDel in parallel on the 24 pieces across 24 cores, and record total wall-clock time alongside the separately measured BWT construction time. The rationale is that sorting groups reads with similar contexts so per-piece BWTs remain informative, and parallelism over pieces yields about one order of magnitude speedup: on the 30x synthetic Human chromosome 1 dataset the parallel pipeline (HARC + BCR + EBWT2INDEL) terminated in 33 minutes versus the sequential pipeline being roughly 10 times slower. Compare the resulting runtime against DISCOSNP++ run with multi-threading to confirm comparable times with higher sensitivity.

## Related tools

- **HARC** (Pre-processing tool that sorts the fasta by context before splitting into 24 pieces) — https://github.com/shubhamchandak94/HARC
- **BCR_LCP_GSA** (Builds the per-piece BWT for each of the 24 sorted fasta pieces (RAM budget in MB, e.g. 1024)) — https://github.com/giovannarosone/BCR_LCP_GSA
- **ebwt2InDel** (Runs in parallel on the 24 BWT pieces to detect SNPs) — github.com/nicolaprezza/ebwt2InDel
- **pebwt2InDel** (Driver script that orchestrates HARC sorting, piece splitting, per-piece BWT construction, and parallel ebwt2InDel execution on 24 cores) — github.com/nicolaprezza/ebwt2InDel
- **DISCOSNP++** (State-of-the-art competitor tool run with multi-threading as the runtime baseline for comparison)
- **SimSeq** (Generates the synthetic 30x-covered Human chromosome 1 read dataset used to benchmark sequential vs parallel runtimes)

## Examples

```
pebwt2InDel <sorted.fasta> 24
```

## Evaluation signals

- Parallel pipeline wall-clock time on the 30x Human chromosome 1 dataset should be about one order of magnitude (≈10x) faster than the sequential pipeline
- Absolute runtime on a 24-core machine should be in the range of the reference point of 33 minutes for HARC + BCR + EBWT2INDEL on 24 pieces
- Parallel runtime should be comparable to DISCOSNP++ run under multi-threading on the same dataset
- BWT construction time must be recorded separately from the SNP-detection wall-clock time when tabulating runtimes
- Parallel variant calls should be validated (e.g. with hap.py) to confirm sensitivity is not degraded relative to the sequential run

## Limitations

- The parallel pipeline at the moment works for SNPs only; INDEL support in the parallel path is deferred to a forthcoming update
- The external pre-processing tool (HARC) does not take into account the presence of INDELs in the read set
- Speedup was demonstrated on a specific 24-core machine and 30x-covered Human chromosome 1 data; results may differ on other hardware or coverage levels

## Evidence

- [results] Parallel SNP-detection pipeline terminated in 33 minutes, ~10x faster than sequential: "The parallel pipeline using HARC followed by BCR and EBWT2INDEL (run in parallel on 24 pieces of the sorted fasta file) terminated in 33 minutes."
- [abstract] Parallel version one order of magnitude faster on 24 cores: "On a 24-cores machine, the parallel version of our tool is one order of magnitude faster than the sequential one"
- [discussion] Parallel pipeline limited to SNPs only: "As a limitation, our parallel pipeline at the moment works for SNPs only"
- [methods] Reads must contain only A, C, G, T: "the DNA fragments of the input read set must contain only symbols 'A', 'C', 'G', 'T'"
- [readme] HARC tool availability: [evidence span withheld — anchor status: external_doc]
