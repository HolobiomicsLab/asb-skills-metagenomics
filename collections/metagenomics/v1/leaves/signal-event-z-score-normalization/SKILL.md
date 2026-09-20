---
name: signal-event-z-score-normalization
description: Use when segmenting raw nanopore FAST5/SLOW5 signals into events and
  you need event means normalized and quantized such that identical DNA content yields
  identical hash values despite signal noise, e.g. for RawHash-style hash-based mapping.
when_to_use_negative:
- Input is already basecalled sequence (FASTQ/FASTA) rather than raw signal — use
  a sequence-based aligner such as minimap2 instead
- Reads are R10.4.1 data for adaptive-sampling-style signal mapping, where legacy
  R9.4 6-mer-model-based approaches like UNCALLED do not apply
- You need per-sample absolute current-level statistics rather than hash-comparable
  normalized/quantized events
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: RawHash
  role: 'Defines the quantization and 32-bit hash-generation pipeline being reimplemented:
    z-score-normalized event means are quantized so equal DNA content yields equal
    hashes, then packed into 32-bit values for the hash table'
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Provides the improved quantization mechanism whose dynamic bucket sizes over
    the normalized signal-value distribution the normalization/quantization step should
    mirror
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Reference streaming signal mapper whose R9.4 6-mer pore model (template_median68pA.model)
    supplies expected event levels for validating normalized/quantized values
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:9f6f22f6ba1780b495a5eeed9b5a75063df3aaa9122e3c693b48c5098b2ffb13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# signal-event-z-score-normalization

## Summary

Z-score normalizes nanopore event means per read and quantizes them into Q-bit values using RawHash's p-bit pruning encoding (E[1,2]E[3+p,Q]), so that signals from the same DNA content map to identical quantized values that can be packed into 32-bit hash values.

## When to use

Use when segmenting raw nanopore FAST5/SLOW5 signals into events and you need event means normalized and quantized such that identical DNA content yields identical hash values despite signal noise, e.g. for RawHash-style hash-based mapping.

## When NOT to use

- Input is already basecalled sequence (FASTQ/FASTA) rather than raw signal — use a sequence-based aligner such as minimap2 instead
- Reads are R10.4.1 data for adaptive-sampling-style signal mapping, where legacy R9.4 6-mer-model-based approaches like UNCALLED do not apply
- You need per-sample absolute current-level statistics rather than hash-comparable normalized/quantized events

## Inputs

- Raw nanopore signal segments (events) from FAST5/POD5/SLOW5/BLOW5 reads, e.g. D1 SARS-CoV-2 R9.4 FAST5 files
- R9.4 6-mer pore model (extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model) for expected event levels
- Quantization parameters Q (bits) and p (pruning bits)

## Outputs

- Per-read z-score-normalized event-mean sequences
- Q-bit quantized event values encoded via p-bit pruning E[1,2]E[3+p,Q]
- 32-bit hash values packed from n consecutive quantized events, suitable for the RawHash hash table

## How to apply

First split the raw R9.4 signal into events with Scrappie-style segmentation (Welch's t-test on adjacent signal windows to detect current-level step changes). Then z-score normalize the event means per read so the normalized distribution of signal values is comparable across reads. Quantize each normalized event mean into Q-bit values using the p-bit pruning encoding E[1,2]E[3+p,Q]; RawHash2's mechanism dynamically arranges the bucket sizes that each signal value is quantized to based on this normalized distribution, which is what guarantees signals from the same DNA content share a quantized value. Finally pack n consecutive quantized events and compress them with a low-collision hash function into a single 32-bit hash value, the terminal output of the signal-to-hash pipeline. Use the legacy R9.4 6-mer pore model (template_median68pA.model) to generate expected event levels for validating quantization and hash correctness.

## Related tools

- **RawHash** (Defines the quantization and 32-bit hash-generation pipeline being reimplemented: z-score-normalized event means are quantized so equal DNA content yields equal hashes, then packed into 32-bit values for the hash table) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Provides the improved quantization mechanism whose dynamic bucket sizes over the normalized signal-value distribution the normalization/quantization step should mirror) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Reference streaming signal mapper whose R9.4 6-mer pore model (template_median68pA.model) supplies expected event levels for validating normalized/quantized values) — https://github.com/haowenz/sigmap

## Evaluation signals

- Every packed 32-bit hash computed from signals derived from the 6-mer model's expected event levels equals the hash computed directly from the k-mer model events
- Two signals corresponding to the same k-mer/DNA content produce identical quantized values after z-score normalization and Q-bit/p-bit pruning
- Segmentation boundaries from Welch's t-test windowing match precomputed expected boundaries on synthetic step-change signals
- Quantized values fall within the Q-bit range implied by the E[1,2]E[3+p,Q] encoding, with the first 2 bits and pruning bits consistent with the spec
- All unit tests pass on D1 SARS-CoV-2 R9.4 FAST5 sample signals with no assertion failures

## Limitations

- Normalization/quantization is tied to a specific pore model (legacy R9.4 6-mer, 180 mV, 450 bps); other pore chemistries need their own models
- RawHash's original quantization is less accurate/slower than RawHash2's dynamically bucketed mechanism; the choice of bucket sizes strongly affects hash sensitivity
- Slight signal variations that survive normalization can still cause hash misses, limiting similarity-search sensitivity

## Evidence

- [abstract] RawHash achieves an accurate hash-based similarity search via an effective quantization of the raw signals such that signals corresponding to the same DNA content have the same quantized value and,: "RawHash achieves an accurate hash-based similarity search via an effective quantization of the raw signals such that signals corresponding to the same DNA content have the same quantized value and,"
- [methods] The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values.: "The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values."
- [intro] In the mapping step, RawHash generates the hash values from raw signals and queries the hash table: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
- [readme] p extern/kmer_models/r9.4_180mv_450bps_6mer/template_median68pA.model: [evidence span withheld — anchor status: external_doc]
