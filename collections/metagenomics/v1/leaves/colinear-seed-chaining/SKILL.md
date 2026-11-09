---
name: colinear-seed-chaining
description: Use when raw nanopore signal seed matches (hash-table lookup hits) must
  be converted into a single mapping position for a read, i.e., when reconstructing
  the RawHash mapping step over seed matches from a hash index like ref.ind.
when_to_use_negative:
- When no reference-genome hash index exists yet — the chaining step requires seed
  matches retrieved from the index built in the indexing step.
- When overlaps between raw signals are needed without a reference genome (all-vs-all
  overlapping) — use Rawsamble instead of reference-guided chaining.
- When base-level alignment of the raw signal is required — chaining yields only approximate
  mapping coordinates, not signal-level DTW alignment.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Performs the chaining over seed matches to map raw signals to the reference
    genome in real time; selects the best colinear chain as the mapped region
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Source of the chaining algorithm reused by RawHash (with the distance coefficient
    disabled, since RawHash does not calculate distances between seed matches)
  repo: https://github.com/haowenz/sigmap
- name: UNCALLED
  role: Defines the PAF output template (together with Sigmap) in which the chain-derived
    mapping positions are reported
  repo: https://github.com/skovaka/UNCALLED
- name: minimap2
  role: Provides ground-truth mapping positions used to validate the chain-derived
    mapping positions
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:01f7a4fb2db28b18a9ef3281a2737a983c8fe517068a460004811855f08e1fcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# colinear-seed-chaining

## Summary

Converts matching seed hash values between a raw nanopore signal and a reference genome into a mapped region by chaining them into the best colinear chain, using Sigmap's chaining algorithm with the distance coefficient disabled.

## When to use

Use when raw nanopore signal seed matches (hash-table lookup hits) must be converted into a single mapping position for a read, i.e., when reconstructing the RawHash mapping step over seed matches from a hash index like ref.ind.

## When NOT to use

- When no reference-genome hash index exists yet — the chaining step requires seed matches retrieved from the index built in the indexing step.
- When overlaps between raw signals are needed without a reference genome (all-vs-all overlapping) — use Rawsamble instead of reference-guided chaining.
- When base-level alignment of the raw signal is required — chaining yields only approximate mapping coordinates, not signal-level DTW alignment.

## Inputs

- Raw nanopore signal reads (FAST5/POD5/SLOW5/BLOW5, e.g., d1_sars-cov-2_r94 FAST5 directory)
- Hash-table index over the reference genome (e.g., ref.ind built from ref.fasta in the indexing step)

## Outputs

- Mapping positions per read in a modified PAF template (mapping.paf, as used by UNCALLED and Sigmap)

## How to apply

Generate seed hashes from each raw signal (e.g., FAST5 reads from a d1_sars-cov-2_r94 dataset) and look up matching genomic positions in the hash-table index (ref.ind) previously built over the reference genome (ref.fasta). Apply the Sigmap-style chaining algorithm over the retrieved seed matches to form colinear chains, with the distance coefficient disabled because RawHash does not calculate distances between seed matches. Select the best-scoring colinear chain per read as the mapped region, following RawHash's real-time behavior of stopping once a read prefix maps. Report the chain-derived mapping positions in the modified PAF template used by UNCALLED and Sigmap.

## Related tools

- **RawHash** (Performs the chaining over seed matches to map raw signals to the reference genome in real time; selects the best colinear chain as the mapped region) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Source of the chaining algorithm reused by RawHash (with the distance coefficient disabled, since RawHash does not calculate distances between seed matches)) — https://github.com/haowenz/sigmap
- **UNCALLED** (Defines the PAF output template (together with Sigmap) in which the chain-derived mapping positions are reported) — https://github.com/skovaka/UNCALLED
- **minimap2** (Provides ground-truth mapping positions used to validate the chain-derived mapping positions) — https://github.com/lh3/minimap2

## Examples

```
RawHash -m -r ref.fasta -i ref.ind -o mapping.paf reads_fast5_dir/
```

## Evaluation signals

- mapping.paf exists and parses as a PAF-formatted table
- Chain-derived mapping positions on an evaluated read subset agree with minimap2-derived ground-truth positions
- Exactly one best-scoring colinear chain is selected per read
- Output PAF follows the modified template used by UNCALLED and Sigmap

## Limitations

- The chaining step does not calculate distances between seed matches (the Sigmap distance coefficient is disabled), so it cannot leverage inter-seed distance scoring.
- Real-time behavior stops once a read prefix maps, which reflects RawHash's prefix-based stopping rule rather than exhaustive read mapping.

## Evidence

- [methods] chaining selects best colinear chain: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [methods] Sigmap and UNCALLED PAF template: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [readme] RawHash real-time mapping purpose: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [intro] mapping step queries hash table: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
