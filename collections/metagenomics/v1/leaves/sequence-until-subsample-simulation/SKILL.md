---
name: sequence-until-subsample-simulation
description: Use when a nanopore raw-signal workflow must be evaluated under truncated
  sequencing (small random fractions of reads) or when quantifying how many reads
  a real-time stopping mechanism like Sequence Until consumes before safely halting.
when_to_use_negative:
- The input is basecalled sequence reads (FASTQ/BAM) rather than raw nanopore signals
  — RawHash and UNCALLED operate on raw current signals
- The data is r10.4.1 rather than legacy r9.4(.1) — UNCALLED can only be applied to
  legacy r9.4.1 data, so the comparison is invalid
- The goal is adaptive-sampling enrichment of targets on-device rather than estimating
  how early a run can be safely terminated from an offline read set
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Maps raw nanopore signals to the reference in real time and provides the Sequence
    Until confidence/outlier stopping mechanism used to terminate processing after
    a read prefix maps
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Builds the reference index (rawhash2 -d ref.ind) using the legacy r9.4 450bps
    6-mer pore model under extern with 32 threads
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline real-time raw-signal mapper against which RawHash's abundance-estimation
    accuracy and read consumption are compared
  repo: https://github.com/skovaka/UNCALLED
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:20a143e5bead0ad8192f1197ec38b72e325f3a313a51f97f11dadb8ae0ad583c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - UNCALLED
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# sequence-until-subsample-simulation

## Summary

Emulates progressively truncated nanopore sequencing runs by generating randomly shuffled FAST5 subsamples (0.01%–25%) of a relative-abundance read set, then evaluates RawHash vs UNCALLED relative-abundance estimation accuracy and RawHash's Sequence Until early-termination read consumption.

## When to use

Use when a nanopore raw-signal workflow must be evaluated under truncated sequencing (small random fractions of reads) or when quantifying how many reads a real-time stopping mechanism like Sequence Until consumes before safely halting.

## When NOT to use

- The input is basecalled sequence reads (FASTQ/BAM) rather than raw nanopore signals — RawHash and UNCALLED operate on raw current signals
- The data is r10.4.1 rather than legacy r9.4(.1) — UNCALLED can only be applied to legacy r9.4.1 data, so the comparison is invalid
- The goal is adaptive-sampling enrichment of targets on-device rather than estimating how early a run can be safely terminated from an offline read set

## Inputs

- Relative-abundance FAST5 read set (raw nanopore signals, legacy r9.4 450bps 6-mer data)
- Reference genome in FASTA format (ref.fasta)
- Legacy r9.4 450bps 6-mer pore model under extern
- Subsample fraction specification (0.01%–25%)

## Outputs

- Randomly shuffled FAST5 subsample files at each fraction (0.01%–25%)
- Reference index file (ref.ind)
- Modified-PAF mapping outputs from RawHash and UNCALLED (with and without Sequence Until)
- CSV files reconstructing Tables 5 and 6 (estimation distances and read-usage fractions)

## How to apply

Generate randomly shuffled subsamples of the relative-abundance FAST5 read set at fractions from 0.01% to 25% to emulate progressively truncated sequencing runs. Build the reference index with rawhash2 (-d ref.ind) using the legacy r9.4 450bps 6-mer pore model under extern with 32 threads, then map each subsample with both RawHash (size-matched presets such as -x sensitive for small genomes and -x fast for large genomes) and UNCALLED, writing modified-PAF outputs. Re-run RawHash's real-time mapping with the confidence/outlier-based stopping mechanism enabled and disabled (with and without Sequence Until), stopping once the read prefix maps to the reference. Compute estimation distances between estimated and true relative abundances and the fraction of each read's signal consumed before stopping, and tabulate results into CSV files reconstructing Tables 5 and 6. The rationale is that RawHash's hash-based mapping on ~0.1% of reads is accurate enough to stop the entire sequencing run after processing only ~7% of reads without substantially sacrificing accuracy.

## Related tools

- **RawHash** (Maps raw nanopore signals to the reference in real time and provides the Sequence Until confidence/outlier stopping mechanism used to terminate processing after a read prefix maps) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Builds the reference index (rawhash2 -d ref.ind) using the legacy r9.4 450bps 6-mer pore model under extern with 32 threads) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline real-time raw-signal mapper against which RawHash's abundance-estimation accuracy and read consumption are compared) — https://github.com/skovaka/UNCALLED

## Examples

```
rawhash2 -d ref.ind -r ref.fasta --kmer-model extern/r9.4_450bps_6mer -t 32 && rawhash -i subsample_0.1pct.fast5 -d ref.ind -x fast -o rawhash_0.1pct.paf
```

## Evaluation signals

- The fraction of reads consumed before Sequence Until stops should approximate the reported ~7% of the entire read set
- RawHash's estimation distances using ~0.1% of reads should be smaller than UNCALLED's distances using the entire read set
- Estimation distances computed from the shuffled subsamples should match the relative ordering reported in Tables 5 and 6
- PAF outputs should follow the modified-PAF template used by UNCALLED for both mappers
- Read-usage fractions per read must be recorded and should show early stopping only after the read prefix maps to the reference

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, so the comparison and stopping evaluation are restricted to r9.4.1 chemistry
- Results are based on randomly shuffled subsamples that emulate truncated runs, not actual device-level early termination
- Accuracy at very small fractions (e.g., 0.01%) may degrade; the reported safe point is ~7% of reads processed with RawHash using 0.1% of reads

## Evidence

- [results] Sequence Until enables accurately stopping the entire sequencing after processing only ~7% of the reads: [evidence span withheld — anchor status: not_found]
- [supplementary] Generate randomly shuffled subsamples: [evidence span withheld — anchor status: not_found]
- [methods] Build the reference index with rawhash2: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [methods] RawHash maps reads from FAST5 files: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [readme] UNCALLED legacy r9.4.1 limitation: [evidence span withheld — anchor status: external_doc]
