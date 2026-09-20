---
name: bitwise-hash-packing-of-quantized-events
description: Use when building a RawHash-style index or mapping pipeline in which
  quantized raw-signal event means must be converted into 32-bit hash values for storage
  in or querying against a hash table.
when_to_use_negative:
- The task is basecalling or base-level alignment — this skill operates purely on
  quantized event levels and does not recover nucleotide sequences
- You need full quantization design itself (e.g., RawHash2's dynamically arranged
  bucket sizes) rather than the downstream bit-packing hash stage
- Signal-to-reference alignment requires base-level resolution (e.g., DTW signal alignment
  as in RawAlign), which this hashing stage cannot provide
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash
  role: Source implementation of the event quantization and 32-bit hash-generation
    pipeline used to build the hash table for real-time raw-signal mapping
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Successor mechanism whose updated quantization (dynamically arranged bucket
    sizes) feeds the same hash-based mapping workflow
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:abcf4b3470b7c2a5b6bea283873c48589bf7b87d6e8d5513df784065844d0657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# bitwise-hash-packing-of-quantized-events

## Summary

Implements the terminal stage of RawHash's signal-to-hash pipeline: compressing the bits of n consecutive quantized nanopore event values (Q-bit trimmed, p-bit pruned) into a single low-collision 32-bit hash value so signals from the same DNA content collide identically.

## When to use

Use when building a RawHash-style index or mapping pipeline in which quantized raw-signal event means must be converted into 32-bit hash values for storage in or querying against a hash table.

## When NOT to use

- The task is basecalling or base-level alignment — this skill operates purely on quantized event levels and does not recover nucleotide sequences
- You need full quantization design itself (e.g., RawHash2's dynamically arranged bucket sizes) rather than the downstream bit-packing hash stage
- Signal-to-reference alignment requires base-level resolution (e.g., DTW signal alignment as in RawAlign), which this hashing stage cannot provide

## Inputs

- Raw nanopore signal values from FAST5 (also POD5, SLOW5, or BLOW5) files
- Quantized event values encoded with Q-bit trimming and p-bit pruning (E[1,2]E[3+p,Q])
- R9.4 6-mer pore model (template_median68pA.model) for expected event levels of the reference genome

## Outputs

- 32-bit hash values packed from n consecutive quantized events
- Hash table entries for reference-genome index generation (e.g., ref.ind) or mapping queries

## How to apply

First segment the raw signal (e.g., from FAST5 files sequenced with R9.4 chemistry) into events using Scrappie-style segmentation based on Welch's t-test over adjacent signal windows, then z-score normalize event means per read and quantize them into Q-bit values using the trimming-and-pruning encoding E[1,2]E[3+p,Q], which makes signals from the same DNA content map to identical quantized values. Pack the bits of n consecutive quantized event values with a low-collision hash function to yield one 32-bit hash value per window of events — this is the terminal output of the signal-to-hash pipeline. The same hashing is applied during indexing, where hash values are generated from the expected signal representation of the reference genome (via the R9.4 6-mer pore model such as extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) and stored in a hash table; during mapping, hashes from raw signals query that table. Validate by asserting that hashes computed from k-mer-model-derived synthetic signals equal those computed from corresponding real FAST5 signal segments.

## Related tools

- **RawHash** (Source implementation of the event quantization and 32-bit hash-generation pipeline used to build the hash table for real-time raw-signal mapping) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Successor mechanism whose updated quantization (dynamically arranged bucket sizes) feeds the same hash-based mapping workflow) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
./rawhash -i -r ref.fasta -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -o ref.ind -t 32
```

## Evaluation signals

- Every packed 32-bit hash computed from reference-derived synthetic signals equals the expected hash computed from k-mer model expected event levels
- Signals corresponding to the same DNA content produce identical quantized values and therefore identical hash values despite slight signal variations
- Unit tests on segmentation boundaries, quantized values, and packed hashes pass on both synthetic signals and D1 SARS-CoV-2 R9.4 FAST5 sample signals
- Hash values generated from raw signals successfully query the index hash table built from the reference genome's expected signal representation

## Limitations

- Hash quality depends entirely on the quantization stage: if event means are not normalized and encoded with Q-bit trimming/p-bit pruning identically across reads, same-DNA signals will hash differently
- RawHash2 replaced the fixed quantization with dynamically arranged bucket sizes based on the normalized signal distribution, indicating the original scheme can be improved in accuracy and performance
- UNCALLED applies only to legacy r9.4.1 data; similar chemistry constraints (R9.4 6-mer model) bound the applicability of the pore-model-derived expected event levels

## Evidence

- [abstract] RawHash achieves accurate hash-based similarity search via effective quantization of raw signals: "RawHash achieves an accurate hash-based similarity search via an effective quantization of the raw signals such that signals corresponding to the same DNA content have the same quantized value and,"
- [intro] Indexing generates hash values from expected signal representation: "The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table."
- [methods] RawHash2 dynamically arranged quantization buckets: "The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values."
- [readme] RawHash is hash-based mapping of raw nanopore signals: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
