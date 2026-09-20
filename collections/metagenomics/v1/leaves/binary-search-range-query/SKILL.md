---
name: binary-search-range-query
description: Use when reconstructing, reimplementing, or validating a minimizer-indexed
  k-mer database (e.g., Kraken-style k-mer/LCA store) and you need exact per-k-mer
  lookup via index-offset range retrieval plus binary search.
when_to_use_negative:
- The classification task requires inexact alignment or sequence similarity (e.g.,
  BLAST/Megablast-style sensitive search) rather than exact-match k-mer queries
- You are classifying raw reads end-to-end (root-to-leaf tree scoring over all k-mers)
  rather than building or testing the database lookup structure itself
- K-mers contain ambiguous nucleotides, which Kraken ignores and which therefore do
  not contribute to classification
edam_operation: http://edamontology.org/operation_0298
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: Kraken
  role: Reference implementation of the minimizer-grouped k-mer/LCA database, index,
    and binary-search query procedure
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter whose database file's 4-byte count spaces are
    repurposed to store LCA taxonomic IDs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:3588fd7f43e1b0e4233da87445bae6bd77c9f29d06c0f03ac710d098f65804ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_0298
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# binary-search-range-query

## Summary

Reconstructs Kraken's k-mer/LCA database structure in which k-mers sharing a 15-bp minimizer are stored consecutively (sorted by canonical representation), with a fixed-size index mapping every 15-mer minimizer value to an offset and length, enabling per-k-mer lookup via index range lookup followed by an in-range binary search.

## When to use

Use when reconstructing, reimplementing, or validating a minimizer-indexed k-mer database (e.g., Kraken-style k-mer/LCA store) and you need exact per-k-mer lookup via index-offset range retrieval plus binary search.

## When NOT to use

- The classification task requires inexact alignment or sequence similarity (e.g., BLAST/Megablast-style sensitive search) rather than exact-match k-mer queries
- You are classifying raw reads end-to-end (root-to-leaf tree scoring over all k-mers) rather than building or testing the database lookup structure itself
- K-mers contain ambiguous nucleotides, which Kraken ignores and which therefore do not contribute to classification

## Inputs

- Library of reference genomic sequences (one taxon each) for k-mer LCA assignment
- Jellyfish k-mer count database file whose 4-byte count spaces will hold LCA taxonomic IDs
- Minimizer length M (15 bp) and k-mer length parameters
- NCBI taxonomy database for LCA computation

## Outputs

- K-mer/LCA database file with k-mers grouped consecutively by minimizer and sorted by canonical representation within groups
- Fixed-size minimizer index mapping each 15-bp minimizer value to an offset and length record
- LCA taxon ID retrieved per query k-mer via binary search in the indexed range

## How to apply

Count k-mers with Jellyfish, then overwrite the 4-byte count spaces with each k-mer's LCA taxonomic ID, computing the LCA of the stored and new taxon on conflicts. Partition records so k-mers sharing a 15-bp (M=15) minimizer are stored consecutively, sorted within each group by canonical k-mer representation; compute the minimizer using lexicographical comparison after XOR-toggling half the bits of each M-mer's canonical representation to even out the minimizer distribution. Build a fixed-size index mapping every possible 15-mer minimizer value to the offset and length of its group's range in the database file. To query a k-mer R, look up the index positions where k-mers with R's minimizer are stored, then binary search within that range to retrieve the LCA taxon ID. Optimize by first searching the previous query's range and skipping minimizer recomputation when the minimizer is unchanged.

## Related tools

- **Kraken** (Reference implementation of the minimizer-grouped k-mer/LCA database, index, and binary-search query procedure) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter whose database file's 4-byte count spaces are repurposed to store LCA taxonomic IDs)

## Evaluation signals

- Query a sample of database k-mers and confirm returned LCA taxon IDs match the source LCA database (output_matches_reference)
- Index file exists and each record contains the expected offset and length fields for its minimizer value (file_exists, field_present)
- k-mers sharing a minimizer are stored consecutively, and within each group records are sorted by canonical k-mer representation
- Binary search of a query k-mer never reads outside the offset/length range specified by the index entry for its minimizer
- Cached-range optimization (searching the previous range first) returns identical taxon IDs to full minimizer recomputation

## Limitations

- The fixed-size index requires a known minimizer alphabet size (all possible 15-mer values); changing M changes the index structure
- MiniKraken's 4-GB compressed database is built by removing the first 18 of every block of 19 records, so only one k-mer per minimizer group survives and group lengths shrink
- K-mers spanning ambiguous nucleotides ('N') are ignored by Kraken and do not affect classification

## Evidence

- [discussion] index range lookup plus binary search: [evidence span withheld — anchor status: not_found]
- [discussion] XOR toggling of minimizer bits: [evidence span withheld — anchor status: not_found]
- [discussion] cached-range optimization: "Rather than compute the minimizer each time we perform a query, we first search the previous range"
- [discussion] database reuse of Jellyfish count spaces: "the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers"
- [readme] Kraken documentation location: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
