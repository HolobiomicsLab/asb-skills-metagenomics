---
name: nanopore-raw-signal-seed-hash-matching
description: Use when mapping raw nanopore signal reads to a reference genome via
  hash-based seed matching, or when reproducing how RawHash converts matching seed
  hash values into a mapped genomic region through colinear chaining.
when_to_use_negative:
- You need base-level or signal-level alignment of the mapped region (RawHash reports
  approximate mapping via chaining, not aligned bases); use a downstream aligner such
  as DTW-based signal alignment instead
- There is no reference genome and the goal is all-vs-all overlap finding between
  raw signals — use Rawsamble instead
- Reads are r10.4.1 data with tools like UNCALLED that only support legacy r9.4.1
  — verify tool compatibility with the pore chemistry
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Generates seed hashes from raw signals, queries the hash-table index, and
    performs Sigmap-style chaining to report mapped regions
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Source of the chaining algorithm over seed matches, with the distance coefficient
    disabled; also co-defines the PAF output template
  repo: https://github.com/haowenz/sigmap
- name: UNCALLED
  role: Co-defines the PAF output template used to report mapping information
  repo: https://github.com/skovaka/UNCALLED
- name: minimap2
  role: Provides minimap2-derived ground-truth mapping positions used to validate
    chain-derived mapping positions
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:ac6556b1e548822489d02e5769e844f986cc014f50bf4849af0531f5b29fe7e5
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
yes_rate: 0.6153846153846154
n_claims: 13
---

# nanopore-raw-signal-seed-hash-matching

## Summary

Reconstructs RawHash's mapping step: seed hashes generated from raw nanopore signals (FAST5/POD5/SLOW5/BLOW5) are looked up in a hash-table index of the reference genome, and the best colinear chain of seed matches — found with Sigmap's chaining algorithm with the distance coefficient disabled — defines the mapped region.

## When to use

Use when mapping raw nanopore signal reads to a reference genome via hash-based seed matching, or when reproducing how RawHash converts matching seed hash values into a mapped genomic region through colinear chaining.

## When NOT to use

- You need base-level or signal-level alignment of the mapped region (RawHash reports approximate mapping via chaining, not aligned bases); use a downstream aligner such as DTW-based signal alignment instead
- There is no reference genome and the goal is all-vs-all overlap finding between raw signals — use Rawsamble instead
- Reads are r10.4.1 data with tools like UNCALLED that only support legacy r9.4.1 — verify tool compatibility with the pore chemistry

## Inputs

- Raw nanopore signal reads (FAST5, POD5, SLOW5, or BLOW5 files, e.g., d1_sars-cov-2_r94 FAST5 directory)
- RawHash hash-table index over the reference genome (ref.ind, built from ref.fasta in the indexing step)
- Reference genome in sequence format (ref.fasta)

## Outputs

- mapping.paf — chain-derived mapping positions per read in the modified PAF template used by UNCALLED and Sigmap

## How to apply

Load the raw nanopore signals (e.g., D1 SARS-CoV-2 r9.4 FAST5 files) and the hash index (ref.ind) previously built over the reference genome (e.g., ref.fasta). For each read, generate seed hashes from the quantized raw signal and look up matching genomic positions in the hash-table index built during the indexing step. Apply Sigmap-style chaining over the retrieved seed matches with the distance coefficient disabled, since RawHash does not calculate distances between seed matches; this yields colinear seed chains. Select the best-scoring colinear chain per read as the mapped region, following RawHash's real-time behavior of stopping once a read prefix maps. Report the chain-derived positions in the modified PAF template shared with UNCALLED and Sigmap.

## Related tools

- **RawHash** (Generates seed hashes from raw signals, queries the hash-table index, and performs Sigmap-style chaining to report mapped regions) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Source of the chaining algorithm over seed matches, with the distance coefficient disabled; also co-defines the PAF output template) — https://github.com/haowenz/sigmap
- **UNCALLED** (Co-defines the PAF output template used to report mapping information) — https://github.com/skovaka/UNCALLED
- **minimap2** (Provides minimap2-derived ground-truth mapping positions used to validate chain-derived mapping positions) — https://github.com/lh3/minimap2

## Examples

```
git clone --recursive https://github.com/CMU-SAFARI/RawHash.git rawhash2 && ./rawhash2/bin/rawhash2 -i ref.ind --fast5 d1_sars-cov-2_r94/ -o mapping.paf
```

## Evaluation signals

- mapping.paf exists and parses as a valid PAF-formatted table
- Chain-derived mapping positions on the evaluated read subset agree with minimap2-derived ground-truth positions
- Each mapped region is the region covered by the best-scoring colinear chain of seed matches for that read
- Chaining was run with the distance coefficient disabled (no distance computation between seed matches), matching RawHash's configuration
- Real-time behavior is preserved: mapping stops once a read prefix maps rather than exhaustively chaining the full signal

## Limitations

- RawHash does not calculate distances between seed matches, so the Sigmap chaining distance coefficient is disabled; chains rely purely on colinearity of hash positions
- Mapping is approximate (chain-derived region), not base-level alignment; accuracy and throughput gains are demonstrated for large genomes but positions are approximate coordinates
- Real-time prefix-stopping means only a read prefix may drive the reported mapping, which can affect accuracy on reads where the prefix is uninformative

## Evidence

- [methods] Chaining with distance coefficient disabled: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [methods] PAF output template: "We follow the similar PAF template used in [UNCALLED]"
- [intro] Mapping step queries hash table: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
- [methods] Index input formats: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [readme] Sigmap PAF template: "and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
