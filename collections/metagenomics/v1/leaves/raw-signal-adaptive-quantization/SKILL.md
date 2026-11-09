---
name: raw-signal-adaptive-quantization
description: Use when you need to convert raw Oxford Nanopore electrical signals (FAST5,
  POD5, SLOW5, BLOW5) into deterministic hash values for real-time hash-based mapping
  or overlap search, and signal-to-signal variability must not change the hash for
  the same DNA content.
when_to_use_negative:
- The task is basecalling or full base-level alignment rather than hash-based raw
  signal mapping (use a basecaller or RawHash's downstream chaining step instead)
- You need base-level CIGAR/PAF alignment output directly from raw signal alignment
  — this skill only produces the upstream 32-bit hashes; chaining over seed matches
  happens afterwards in RawHash
- Data is not nanopore raw electrical current signal (e.g., Illumina FASTQ input)
tools:
- name: RawHash
  role: Reference implementation of the raw-signal-to-hash pipeline (event generation,
    quantization, 32-bit hashing) used for hash-table indexing and mapping
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Improved variant whose new quantization mechanism dynamically arranges bucket
    sizes from the normalized signal-value distribution
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Comparison baseline (with Sigmap) against which RawHash's quantization/hashing
    throughput and accuracy are judged
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Comparison baseline for raw-signal mapping that also consumes 6-mer pore models
    for indexing
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:7b446606c742c33269fa775957074c515a64ae28c68ada6dcf4ede77058476ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# raw-signal-adaptive-quantization

## Summary

Implements RawHash's raw-signal-to-hash pipeline for nanopore data: raw FAST5/BLOW5 signals are segmented into events (Scrappie-style Welch's t-test segmentation), z-score normalized, adaptively quantized (Q-bit trimming with p-bit pruning encoding E[1,2]E[3+p,Q]) so that signals from the same DNA content yield identical quantized values, and packed n consecutive events into a single 32-bit hash value.

## When to use

Use when you need to convert raw Oxford Nanopore electrical signals (FAST5, POD5, SLOW5, BLOW5) into deterministic hash values for real-time hash-based mapping or overlap search, and signal-to-signal variability must not change the hash for the same DNA content.

## When NOT to use

- The task is basecalling or full base-level alignment rather than hash-based raw signal mapping (use a basecaller or RawHash's downstream chaining step instead)
- You need base-level CIGAR/PAF alignment output directly from raw signal alignment — this skill only produces the upstream 32-bit hashes; chaining over seed matches happens afterwards in RawHash
- Data is not nanopore raw electrical current signal (e.g., Illumina FASTQ input)

## Inputs

- Raw nanopore signal files (FAST5, POD5, SLOW5, or BLOW5)
- R9.4 6-mer pore model (e.g., extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model)
- Synthetic raw signals generated from the k-mer model's expected event levels (for unit tests)

## Outputs

- Per-read event segmentation boundaries (step-change positions detected by Welch's t-test)
- Q-bit quantized event values after z-score normalization and p-bit pruning
- 32-bit hash values packed from n consecutive quantized events, fed into the RawHash hash table

## How to apply

Load the legacy R9.4 6-mer pore model (template_median68pA.model) and read raw signals from FAST5 files. Segment each raw signal into events with Scrappie-style event detection that applies Welch's t-test to adjacent signal windows to locate step changes in current level. Z-score normalize the per-read event means and quantize them so signals corresponding to the same DNA content collapse to the same quantized value — RawHash2's newer mechanism dynamically arranges the bucket sizes based on the normalized distribution of signal values. Finally, a low-collision hash function compresses the bits of n consecutive quantized event values into a single 32-bit hash, the terminal output used to populate and query RawHash's hash table. Validate by asserting that hashes generated from signals synthesized from k-mer-model expected event levels equal the hashes computed directly from those model events.

## Related tools

- **RawHash** (Reference implementation of the raw-signal-to-hash pipeline (event generation, quantization, 32-bit hashing) used for hash-table indexing and mapping) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Improved variant whose new quantization mechanism dynamically arranges bucket sizes from the normalized signal-value distribution) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Comparison baseline (with Sigmap) against which RawHash's quantization/hashing throughput and accuracy are judged) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Comparison baseline for raw-signal mapping that also consumes 6-mer pore models for indexing) — https://github.com/haowenz/sigmap

## Evaluation signals

- All unit tests pass: segmentation boundaries, quantized values, and packed hashes on synthetic signals match precomputed expected values
- Every 32-bit hash from signals derived from the 6-mer model's expected event levels equals the hash computed directly from those k-mer model events
- No assertion failures on D1 SARS-CoV-2 R9.4 FAST5 sample signals (test/data/d1_sars-cov-2_r94/fast5_files)
- Invariance check: signals corresponding to the same DNA content produce the same quantized value and same hash despite slight signal variations
- Hashes generated from raw signals in the mapping step hit the same hash-table entries as hashes pre-generated from the reference genome's expected signal representation

## Limitations

- Quantization is reference-model specific: the legacy R9.4 6-mer model (180 mV, 450 bps) is required and results are tied to that chemistry
- Segmentation via Welch's t-test on adjacent windows can misplace event boundaries under noisy step changes, which propagates into quantized values and hashes
- RawHash is designed for real-time hash-based mapping for large genomes; low-collision packing of n events trades sensitivity for throughput compared to per-base alignment approaches

## Evidence

- [abstract] RawHash achieves an accurate hash-based similarity search via an effective quantization of the raw signals such that signals corresponding to the same DNA content have the same quantized value and,: "RawHash achieves an accurate hash-based similarity search via an effective quantization of the raw signals such that signals corresponding to the same DNA content have the same quantized value and,"
- [methods] The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values.: "The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values."
- [intro] In the mapping step, RawHash generates the hash values from raw signals and queries the hash table: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
- [intro] The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table.: "The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table."
- [readme] RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time.: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
