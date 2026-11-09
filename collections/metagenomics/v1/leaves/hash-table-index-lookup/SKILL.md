---
name: hash-table-index-lookup
description: Use when converting matching seed hash values between a raw nanopore
  signal (FAST5/POD5/SLOW5/BLOW5) and a reference genome into a mapped region via
  hash-table lookup followed by colinear chaining of seed matches.
when_to_use_negative:
- No reference genome is available and all-vs-all overlaps between raw signals are
  needed — use Rawsamble's overlapping mechanism instead.
- Base-level or high-accuracy end-to-end alignment of sequence reads is required —
  use a sequence mapper such as minimap2 rather than raw-signal hash lookup.
- Input is already assembled/sequence reads rather than raw electrical signals — the
  signal-quantization hash lookup does not apply.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash
  role: Generates seed hashes from raw signals, queries the hash-table index, and
    performs chaining over seed matches to map reads to the reference genome
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Source of the chaining algorithm (with distance coefficient disabled) used
    to form colinear seed chains and select the mapped region
  repo: https://github.com/haowenz/sigmap
- name: UNCALLED
  role: Provides the PAF output template used to report chain-derived mapping positions
  repo: https://github.com/skovaka/UNCALLED
- name: minimap2
  role: Produces ground-truth sequence-based mapping positions for validating the
    chain-derived signal mapping
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:c25c5e92aa6c7a091faa8a76fc8b5d7efdfd4a395ba8cb61115c8b46498b871a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hash-table-index-lookup

## Summary

Reconstructs RawHash's mapping step: seed hashes generated from raw nanopore signals are looked up in a hash-table index built from the reference genome's expected signal representation, and matching seed hash positions are chained (Sigmap-style, distance coefficient disabled) into the colinear chain that defines the mapped region.

## When to use

Use when converting matching seed hash values between a raw nanopore signal (FAST5/POD5/SLOW5/BLOW5) and a reference genome into a mapped region via hash-table lookup followed by colinear chaining of seed matches.

## When NOT to use

- No reference genome is available and all-vs-all overlaps between raw signals are needed — use Rawsamble's overlapping mechanism instead.
- Base-level or high-accuracy end-to-end alignment of sequence reads is required — use a sequence mapper such as minimap2 rather than raw-signal hash lookup.
- Input is already assembled/sequence reads rather than raw electrical signals — the signal-quantization hash lookup does not apply.

## Inputs

- Raw nanopore signal reads (FAST5, POD5, SLOW5, or BLOW5; e.g., d1_sars-cov-2_r94 FAST5 directory)
- Hash-table index built over the reference genome (ref.ind from ref.fasta)
- ONT k-mer pore model (e.g., extern/kmer_models/r9.4_180mv_450bps_6mer/template_median68pA.model)

## Outputs

- Mapped region per read: chain-derived mapping positions in the modified PAF template used by UNCALLED and Sigmap (mapping.paf)

## How to apply

First build the hash index by generating hash values from the expected signal representation of the reference genome (e.g., ref.fasta -> ref.ind using a k-mer model such as the r9.4 450bps 6-mer pore model) and storing them in a hash table. In the mapping step, for each raw signal read, generate seed hashes from the quantized signal and query the hash table to retrieve matching genomic positions. Apply the Sigmap chaining algorithm over the retrieved seed matches with the distance coefficient disabled, since RawHash does not calculate distances between seed matches; select the best-scoring colinear chain per read as the mapped region, stopping once a read prefix maps to mimic real-time behavior. Report positions in the PAF template used by UNCALLED and Sigmap.

## Related tools

- **RawHash** (Generates seed hashes from raw signals, queries the hash-table index, and performs chaining over seed matches to map reads to the reference genome) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Source of the chaining algorithm (with distance coefficient disabled) used to form colinear seed chains and select the mapped region) — https://github.com/haowenz/sigmap
- **UNCALLED** (Provides the PAF output template used to report chain-derived mapping positions) — https://github.com/skovaka/UNCALLED
- **minimap2** (Produces ground-truth sequence-based mapping positions for validating the chain-derived signal mapping) — https://github.com/lh3/minimap2

## Examples

```
./rawhash -i ref.ind -f d1_sars-cov-2_r94/ -o mapping.paf
```

## Evaluation signals

- mapping.paf exists and parses as a PAF-formatted table with the UNCALLED/Sigmap template columns.
- Chain-derived mapping positions on the evaluated read subset agree with minimap2-derived ground-truth positions.
- Each read yields a single best-scoring colinear chain selected with the Sigmap distance coefficient disabled.
- Retrieved seed-match genomic positions are consistent with hash entries generated from the reference genome's expected signal representation (ref.ind).

## Limitations

- The chaining step does not calculate distances between seed matches (distance coefficient disabled), so chain scoring is not distance-aware.
- Mapping is approximate, without base-level alignment; coordinates come only from the chain of seed matches.
- Accuracy depends on effective quantization so signals corresponding to the same DNA content hash identically; quantization failures degrade lookup specificity.

## Evidence

- [intro] The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table.: "The indexing step generates hash values from the expected signal representation of a reference genome and stores them in a hash table."
- [intro] Mapping step generates hashes from raw signals and queries the hash table: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
- [methods] RawHash maps raw signal via chaining over seed matches: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [methods] PAF template follows UNCALLED and Sigmap output: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [results] Sigmap chaining is applied with the distance coefficient disabled: [evidence span withheld — anchor status: not_found]
