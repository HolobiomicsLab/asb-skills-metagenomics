---
name: canonical-kmer-representation-xor-scrambling
description: 'Reference for implementing or reproducing Kraken-style minimizer indexing:
  when building a k-mer/LCA database that groups k-mers by 15-bp minimizers and needs
  a fast index-lookup-plus-binary-search query path.'
when_to_use_negative:
- When the input k-mers contain ambiguous nucleotides — Kraken ignores k-mers with
  ambiguous nucleotides, so they do not participate in classification
- When alignment-based sensitivity is required for sequences with no exact k-mer matches
  — exact-match database queries, not inexact alignment, underpin this skill
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: Kraken
  role: Implements the XOR-toggled minimizer computation, minimizer-grouped k-mer
    database, fixed-size index, and index-lookup-plus-binary-search query for per-k-mer
    LCA retrieval
  repo: http://ccb.jhu.edu/software/kraken/
- name: Jellyfish
  role: Multithreaded k-mer counter whose 4-byte count spaces are repurposed to store
    each k-mer's LCA taxonomic ID in the database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:6acea191e3249341c6d3cd6a583dc85e98fd162f8cda7a3697c533959d83ad9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# canonical-kmer-representation-xor-scrambling

## Summary

Kraken's technique for computing evenly distributed minimizers: the canonical representation of each M-mer (M=15) is XOR-toggled on half its bits before lexicographical comparison, so that k-mer records grouped by minimizer form balanced blocks that support index lookup plus binary search for per-k-mer LCA queries.

## When to use

Reference for implementing or reproducing Kraken-style minimizer indexing: when building a k-mer/LCA database that groups k-mers by 15-bp minimizers and needs a fast index-lookup-plus-binary-search query path.

## When NOT to use

- When the input k-mers contain ambiguous nucleotides — Kraken ignores k-mers with ambiguous nucleotides, so they do not participate in classification
- When alignment-based sensitivity is required for sequences with no exact k-mer matches — exact-match database queries, not inexact alignment, underpin this skill

## Inputs

- Reference genome library sequences processed with Jellyfish k-mer counting (4-byte count spaces later overwritten with LCA taxonomic IDs)
- NCBI taxonomy database for computing LCA values of shared k-mers
- Query nucleotide sequences (e.g., 100 bp reads) to be classified k-mer by k-mer

## Outputs

- Kraken k-mer/LCA database file with k-mers grouped consecutively by minimizer and sorted by canonical representation within each group
- Fixed-size minimizer index mapping each 15-bp minimizer value to offset and length of its group's range in the database
- Per-k-mer LCA taxon IDs retrieved via index lookup plus binary search

## How to apply

Compute the canonical representation of each 15-bp (M=15) minimizer, then apply the exclusive-or (XOR) operation to toggle half of the bits prior to lexicographical comparison of the M-mers; this evens out the minimizer distribution so groups of k-mers sharing a minimizer are neither degenerate nor oversized. Store k-mer records so that k-mers sharing a minimizer appear consecutively, sorted within each group by canonical representation, and build a fixed-size index mapping every possible 15-bp minimizer value to its group's offset and length in the database file (LCA taxon IDs are held in the 4-byte spaces Jellyfish used for k-mer counts). To query a k-mer R, look up the index positions where k-mers with R's minimizer would be stored, then binary search within that range to retrieve the LCA taxon ID. As an optimization, first search the previous query's range and skip minimizer recomputation when the minimizer is unchanged, since adjacent k-mers in a read often share a minimizer.

## Related tools

- **Kraken** (Implements the XOR-toggled minimizer computation, minimizer-grouped k-mer database, fixed-size index, and index-lookup-plus-binary-search query for per-k-mer LCA retrieval) — http://ccb.jhu.edu/software/kraken/
- **Jellyfish** (Multithreaded k-mer counter whose 4-byte count spaces are repurposed to store each k-mer's LCA taxonomic ID in the database)

## Evaluation signals

- Query a sample of database k-mers and confirm returned LCA taxon IDs match the source LCA database
- Confirm the index file exists with the expected structure: offset and length fields present for minimizer-group records
- Verify k-mers sharing a minimizer are stored consecutively and sorted within each group by canonical representation
- Check minimizer distribution is evened out (XOR toggling applied) so no minimizer group dominates the index
- Confirm the cached-range optimization returns the same LCA IDs as full minimizer recomputation on the same queries

## Limitations

- Kraken ignores k-mers with ambiguous nucleotides, so k-mers spanning 'N' characters do not affect classification
- The database search relies on exact k-mer matches rather than inexact alignment, so it cannot detect similarity that exact k-mer matching misses

## Evidence

- [discussion] XOR toggling for even minimizer distribution: [evidence span withheld — anchor status: not_found]
- [discussion] Index lookup plus binary search query procedure: [evidence span withheld — anchor status: not_found]
- [discussion] Cached previous-range optimization: "Rather than compute the minimizer each time we perform a query, we first search the previous range"
- [discussion] LCA values stored in Jellyfish count spaces: "the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers"
- [readme] Kraken project documentation: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
