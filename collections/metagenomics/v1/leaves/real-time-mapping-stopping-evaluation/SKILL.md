---
name: real-time-mapping-stopping-evaluation
description: Use when a task asks to quantify estimation accuracy of relative abundance
  from truncated or subsampled nanopore signal read sets, or to measure the fraction
  of reads a real-time mapper consumes before safe early run termination (Sequence
  Until).
when_to_use_negative:
- Input is basecalled sequence reads (FASTQ/FASTA) rather than raw nanopore signal
  files (FAST5/POD5/SLOW5/BLOW5) — RawHash operates on raw signals
- Data is r10.4.1 rather than legacy r9.4 data, since the UNCALLED baseline used here
  can only be applied to legacy r9.4.1 data
- The goal is de novo assembly or all-vs-all overlapping without a reference genome
  — Rawsamble, not this relative-abundance/stopping evaluation, is the appropriate
  workflow
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: RawHash
  role: Hash-based mapper of raw nanopore signals to a reference genome; its real-time
    Sequence Until confidence/outlier stopping mechanism enables early termination
    and accurate abundance estimation from few reads
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: More sensitive and faster raw signal mapping mechanism used to build the reference
    index (ref.ind) with the r9.4 450bps 6-mer pore model
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: State-of-the-art baseline real-time raw signal mapper used for comparison
    of relative abundance estimation accuracy against RawHash
  repo: https://github.com/skovaka/UNCALLED
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:83cd255b2b6248fe60ef529b9b0112d772737839100494c3edd23c5d1fb9e206
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - UNCALLED
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# real-time-mapping-stopping-evaluation

## Summary

A protocol for evaluating how accurately raw nanopore signal mappers (RawHash, UNCALLED) estimate relative abundance from small randomly shuffled read fractions (0.01%–25%) and for measuring how early RawHash's real-time Sequence Until mechanism can safely stop sequencing while preserving accuracy.

## When to use

Use when a task asks to quantify estimation accuracy of relative abundance from truncated or subsampled nanopore signal read sets, or to measure the fraction of reads a real-time mapper consumes before safe early run termination (Sequence Until).

## When NOT to use

- Input is basecalled sequence reads (FASTQ/FASTA) rather than raw nanopore signal files (FAST5/POD5/SLOW5/BLOW5) — RawHash operates on raw signals
- Data is r10.4.1 rather than legacy r9.4 data, since the UNCALLED baseline used here can only be applied to legacy r9.4.1 data
- The goal is de novo assembly or all-vs-all overlapping without a reference genome — Rawsamble, not this relative-abundance/stopping evaluation, is the appropriate workflow

## Inputs

- Relative-abundance FAST5 raw nanopore signal read set
- Reference genome in sequence format (ref.fasta)
- Legacy r9.4 450bps 6-mer pore model (k-mer model under extern)
- Subsample fractions ranging from 0.01% to 25% of reads

## Outputs

- Modified-PAF mapping outputs from RawHash and UNCALLED
- CSV tables of estimation distances (reconstructing Tables 5 and 6)
- Per-read fraction of signal consumed before Sequence Until stopping

## How to apply

Generate randomly shuffled subsamples of the relative-abundance FAST5 read set at fractions from 0.01% to 25% to emulate progressively truncated sequencing runs. Build the reference index with rawhash2 (-d ref.ind) using the legacy r9.4 450bps 6-mer pore model located under extern with 32 threads. Map each subsample against the index with both RawHash and UNCALLED (RawHash using size-matched presets, e.g. -x sensitive for small genomes and -x fast for large genomes), producing modified-PAF outputs. Re-run RawHash's real-time mapping with the confidence/outlier stopping mechanism enabled and disabled (with and without Sequence Until), stopping once the read prefix maps to the reference. Compute estimation distances between estimated and true relative abundances and the fraction of each read's signal consumed before stopping, then tabulate into CSV files reconstructing Tables 5 and 6.

## Related tools

- **RawHash** (Hash-based mapper of raw nanopore signals to a reference genome; its real-time Sequence Until confidence/outlier stopping mechanism enables early termination and accurate abundance estimation from few reads) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (More sensitive and faster raw signal mapping mechanism used to build the reference index (ref.ind) with the r9.4 450bps 6-mer pore model) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (State-of-the-art baseline real-time raw signal mapper used for comparison of relative abundance estimation accuracy against RawHash) — https://github.com/skovaka/UNCALLED

## Examples

```
rawhash2 -d ref.ind -r ref.fasta -p extern/r9.4_450bps_6mer -t 32 && ./rawhash2 -x fast -i reads.fast5 -r ref.ind --sequence-until > mappings.paf
```

## Evaluation signals

- The fraction of reads consumed before Sequence Until stopping should approximate the reported ~7% of the entire read set
- Estimation distances for RawHash versus UNCALLED should match the relative ordering reported in Tables 5 and 6 (RawHash using just 0.1% of reads should be more accurate than UNCALLED using the entire read set)
- Mapping outputs are produced in the modified-PAF template format consistent with UNCALLED's PAF output
- Subsampled accuracy should remain stable without substantial sacrifice across fractions from 0.01% up to 25%
- Index file ref.ind is generated successfully with rawhash2 using 32 threads and the legacy r9.4 450bps 6-mer pore model

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, constraining the comparison to that chemistry/pore model
- Accuracy at very low subsample fractions (down to 0.01%) may vary; the mechanism is validated against a fixed read set rather than live sequencing
- RawHash's minimizer seeding mode is around 3X faster but slightly less accurate than non-minimizer mode, which may affect estimation distance results

## Evidence

- [results] Sequence Until enables accurately stopping the entire sequencing after processing only ~7% of the reads: [evidence span withheld — anchor status: not_found]
- [results] RawHash estimations with few reads beat UNCALLED: [evidence span withheld — anchor status: not_found]
- [methods] RawHash reads FAST5/POD5/SLOW5/BLOW5: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [readme] Indexing example from README: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [readme] UNCALLED restricted to r9.4.1 data: [evidence span withheld — anchor status: external_doc]
