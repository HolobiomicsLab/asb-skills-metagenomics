---
name: cross-tool-computational-performance-comparison
description: Use when reproducing or extending a runtime/speedup claim, e.g. quantifying
  how much faster a parallel pipeline is than its sequential version and whether it
  matches a tool's runtime on the same input dataset.
when_to_use_negative:
- When INDEL detection throughput is the benchmark target — the parallel HARC-based
  pipeline works for SNPs only
- When datasets differ between the candidate and competitor runs — runtime comparison
  is only valid on identical inputs and hardware
- When comparing space efficiency rather than speed — ebwt2InDel's headline improvement
  is memory, measured separately from wall-clock time
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: SimSeq
  role: Generates the synthetic Human chromosome 1 read benchmark at 30x coverage
    with the HiSeq error profile, shared by both pipelines and the competitor
- name: BCR_LCP_GSA
  role: Builds the eBWT of the input reads (sequential run) and per-piece BWTs (parallel
    run); BWT construction time is recorded separately from SNP detection
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: ebwt2InDel
  role: SNP-detection engine whose sequential and parallel runtimes are the comparison
    targets
  repo: github.com/nicolaprezza/ebwt2InDel
- name: HARC
  role: Pre-processing step in the parallel pipeline that sorts the fasta by context
    before splitting into 24 pieces
  repo: https://github.com/shubhamchandak94/HARC
- name: pebwt2InDel
  role: 'Script orchestrating the parallel pipeline: HARC sort, 24-way fasta split,
    per-piece BCR BWTs, and parallel ebwt2InDel execution on 24 cores'
  repo: github.com/nicolaprezza/ebwt2InDel
- name: DISCOSNP++
  role: State-of-the-art competitor whose multi-threaded runtime provides the external
    performance reference point
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:0a2232809446a71e8abe8c4d7ce0feb6db545050030556a62e9fca07376de43c
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
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-tool-computational-performance-comparison

## Summary

Benchmarks a candidate pipeline's wall-clock runtime against both a sequential baseline of the same tool and a state-of-the-art competitor (here, ebwt2InDel's parallel vs sequential pipelines and DISCOSNP++) on a shared dataset to demonstrate speedup and parity claims.

## When to use

Use when reproducing or extending a runtime/speedup claim, e.g. quantifying how much faster a parallel pipeline is than its sequential version and whether it matches a state-of-the-art tool's runtime on the same input dataset.

## When NOT to use

- When INDEL detection throughput is the benchmark target — the parallel HARC-based pipeline works for SNPs only
- When datasets differ between the candidate and competitor runs — runtime comparison is only valid on identical inputs and hardware
- When comparing space efficiency rather than speed — ebwt2InDel's headline improvement is memory, measured separately from wall-clock time

## Inputs

- Simulated or real raw-read dataset in FASTA/FASTQ (e.g. synthetic Human chromosome 1, 30x coverage, HiSeq error profile)
- 24-core machine with sufficient RAM for BCR_LCP_GSA (RAM budget in MB, e.g. 1024)
- Competitor tool executable (DISCOSNP++) configured for multi-threading

## Outputs

- Wall-clock runtime table: sequential pipeline vs parallel pipeline vs competitor (DISCOSNP++)
- Separately recorded BWT-construction time and SNP-detection time for the sequential run
- SNP output files (e.g. output.snp from ebwt2InDel) supporting runtime claims

## How to apply

Generate or select a fixed benchmark dataset — in the reference work, a synthetic Human chromosome 1 read set simulated at 30x coverage with SimSeq using the HiSeq error profile, coverage split uniformly between the two chromosome variants. Run the sequential pipeline first: build the BWT with BCR_LCP_GSA with an explicit RAM budget in MB (e.g. 1024), then run 'ebwt2InDel -1 <bwt> -o output.snp' in single-sample mode, recording the SNP-detection wall-clock time separately from BWT construction time. Then run the parallel pipeline via the pebwt2InDel script, which sorts the fasta by context with HARC, splits the sorted fasta into 24 pieces, builds per-piece BWTs with BCR_LCP_GSA, and runs ebwt2InDel on 24 cores, recording total wall-clock time. Tabulate both runtimes alongside the competitor's (DISCOSNP++) multi-threaded runtime on the same machine and dataset. Rationale: the reference result is a ~33 minute parallel run on the 24-core machine, approximately 10x faster than the complete sequential pipeline, while remaining comparable to DISCOSNP++ in time and more sensitive.

## Related tools

- **SimSeq** (Generates the synthetic Human chromosome 1 read benchmark at 30x coverage with the HiSeq error profile, shared by both pipelines and the competitor)
- **BCR_LCP_GSA** (Builds the eBWT of the input reads (sequential run) and per-piece BWTs (parallel run); BWT construction time is recorded separately from SNP detection) — https://github.com/giovannarosone/BCR_LCP_GSA
- **ebwt2InDel** (SNP-detection engine whose sequential and parallel runtimes are the comparison targets) — github.com/nicolaprezza/ebwt2InDel
- **HARC** (Pre-processing step in the parallel pipeline that sorts the fasta by context before splitting into 24 pieces) — https://github.com/shubhamchandak94/HARC
- **pebwt2InDel** (Script orchestrating the parallel pipeline: HARC sort, 24-way fasta split, per-piece BCR BWTs, and parallel ebwt2InDel execution on 24 cores) — github.com/nicolaprezza/ebwt2InDel
- **DISCOSNP++** (State-of-the-art competitor whose multi-threaded runtime provides the external performance reference point)

## Examples

```
pebwt2InDel --sorted-fasta reads.sorted.fa --pieces 24 --cores 24   # sequential baseline: ebwt2InDel -1 reads.bwt -o output.snp
```

## Evaluation signals

- Parallel pipeline wall-clock time is approximately one order of magnitude (about 10x) lower than the sequential pipeline on the same dataset and machine
- Reference point: parallel pipeline (HARC + BCR + EBWT2INDEL on 24 pieces) terminates in about 33 minutes on the 30x-covered Human chromosome 1 dataset on a 24-core machine
- BWT-construction time is reported separately from SNP-detection time so runtimes are attributable
- Parallel pipeline total runtime is comparable to DISCOSNP++ multi-threaded runtime on the same data
- Both pipelines ran on identically simulated input (30x coverage, HiSeq error profile, uniform coverage split between two chromosome variants)

## Limitations

- The parallel pipeline at the moment works for SNPs only; INDEL support is deferred to a forthcoming update
- The external pre-processing tool (HARC) does not take into account the presence of INDELs in the read set
- Speedup factor and parity with DISCOSNP++ are machine- and dataset-specific (24 cores, 30x Human chromosome 1); no software version numbers are stated for the tools being compared

## Evidence

- [results] Parallel pipeline terminated in 33 minutes, ~10x faster than sequential and comparable to DISCOSNP++: "The parallel pipeline using HARC followed by BCR and EBWT2INDEL (run in parallel on 24 pieces of the sorted fasta file) terminated in 33 minutes."
- [abstract] Parallel version one order of magnitude faster on 24 cores: "On a 24-cores machine, the parallel version of our tool is one order of magnitude faster than the sequential one"
- [results] SimSeq simulation protocol for the shared benchmark dataset: [evidence span withheld — anchor status: not_found]
- [results] BWT construction time separated from SNP-detection time: "We separate the running time required to build the BWT (by using BCR [42, 43]) from the running time of EBWT2INDEL"
- [discussion] Parallel pipeline limitation: SNPs only: "As a limitation, our parallel pipeline at the moment works for SNPs only"
