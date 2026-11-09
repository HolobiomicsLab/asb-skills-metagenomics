---
name: kmer-minimizer-selection
description: Use when building or reimplementing a k-mer/LCA lookup database that
  must support fast per-k-mer queries — e.g. reconstructing Kraken's database where
  k-mers sharing a 15-bp minimizer are stored consecutively and searched via index
  lookup plus binary search.
when_to_use_negative:
- Sequences requiring inexact alignment sensitivity (e.g. reads with divergence not
  captured by exact 31-mer matches) — the article positions this only for exact-match
  k-mer querying
- K-mers containing ambiguous nucleotides — Kraken ignores k-mers with ambiguous nucleotides,
  so 'N'-spanning k-mers do not affect classification
- Analyses where database memory footprint must be very small without lossy sampling
  trade-offs — the MiniKraken 4-GB database achieves this by removing the first 18
  of every block of 19 records, degrading information
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Kraken
  role: Implements the minimizer-grouped k-mer/LCA database with index-plus-binary-search
    lookup for taxonomic classification of metagenomic sequences
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter whose 4-byte count spaces are reused to store
    each k-mer's LCA taxonomic ID in the database file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:26f78f52ef6aba8cb57c11c7649c8cba703a1384f84963838a1030b9661ace8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# kmer-minimizer-selection

## Summary

Selects minimizers (Kraken uses M=15 canonical M-mers with XOR bit-toggling) to group k-mer/LCA records so that a fixed-size index plus in-range binary search yields fast exact k-mer taxon lookup. This is the database-organization scheme underlying Kraken's exact-match k-mer classification speed.

## When to use

Use when building or reimplementing a k-mer/LCA lookup database that must support fast per-k-mer queries — e.g. reconstructing Kraken's database where k-mers sharing a 15-bp minimizer are stored consecutively and searched via index lookup plus binary search.

## When NOT to use

- Sequences requiring inexact alignment sensitivity (e.g. reads with divergence not captured by exact 31-mer matches) — the article positions this only for exact-match k-mer querying
- K-mers containing ambiguous nucleotides — Kraken ignores k-mers with ambiguous nucleotides, so 'N'-spanning k-mers do not affect classification
- Analyses where database memory footprint must be very small without lossy sampling trade-offs — the MiniKraken 4-GB database achieves this by removing the first 18 of every block of 19 records, degrading information

## Inputs

- K-mer/LCA database produced by counting k-mers with Jellyfish and overwriting the 4-byte count spaces with LCA taxonomic IDs
- Nucleotide reference sequences with taxonomic assignments (e.g. NCBI RefSeq complete microbial genomes; taxon information from the NCBI taxonomy database)
- Query k-mers or reads to be classified

## Outputs

- Minimizer-partitioned database file with k-mer records grouped by minimizer and sorted by canonical representation within each group
- Fixed-size index file mapping each 15-bp minimizer value to the offset and length of its group's range
- LCA taxon ID(s) retrieved per query k-mer

## How to apply

Load the k-mer LCA database in which Jellyfish's 4-byte count spaces hold each k-mer's LCA taxonomic ID instead of counts. For each k-mer compute the canonical representation of its 15-bp (M=15) minimizer, applying the exclusive-or (XOR) operation to toggle half the bits of each M-mer's canonical representation before lexicographical comparison so the minimizer distribution is evened out. Partition and store records so k-mers sharing a minimizer appear consecutively, sorted within each group by canonical representation, then build a fixed-size index mapping every possible 15-bp minimizer value to the offset and length of its group's range in the database file. To query a k-mer R, look up the index positions where k-mers with R's minimizer are stored and perform a binary search within that range to retrieve the LCA taxon ID; apply the cached-range optimization by first searching the previous query's range and skipping minimizer recomputation when the minimizer is unchanged. This exact-match structure (rather than inexact alignment) is what lets Kraken classify 100 bp reads at over 4.1 million reads per minute.

## Related tools

- **Kraken** (Implements the minimizer-grouped k-mer/LCA database with index-plus-binary-search lookup for taxonomic classification of metagenomic sequences) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter whose 4-byte count spaces are reused to store each k-mer's LCA taxonomic ID in the database file)

## Evaluation signals

- Query a sample of database k-mers and confirm returned LCA taxon IDs match the source LCA database (output_matches_reference)
- Confirm the index file exists with the expected structure (file_exists, field_present on offset/length records)
- Verify k-mers sharing a minimizer are stored consecutively and sorted within each group by canonical representation
- Verify XOR bit-toggling produces an even minimizer distribution across the 15-bp minimizer space
- Confirm the cached-range optimization is exercised: consecutive queries with the same minimizer skip minimizer recomputation and search the previous range first

## Limitations

- The fixed-size index covers every possible 15-bp minimizer value, so a different M requires rebuilding both the index and the record ordering
- MiniKraken (4 GB) discards records — the first 18 of every block of 19 — trading classification power for memory
- Kraken ignores k-mers with ambiguous nucleotides, so k-mers spanning 'N' characters do not affect classification

## Evidence

- [discussion] A query for a k-mer R can then be processed by looking up in an index the positions in the database where the k-mers with R's minimizer would be stored: [evidence span withheld — anchor status: not_found]
- [discussion] we use the exclusive-or (XOR) operation to toggle half of the bits of each M-mer's canonical representation prior to comparing the M-mers: [evidence span withheld — anchor status: not_found]
- [discussion] Rather than compute the minimizer each time we perform a query, we first search the previous range: "Rather than compute the minimizer each time we perform a query, we first search the previous range"
- [discussion] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers: [evidence span withheld — anchor status: not_found]
- [readme] Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken.: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
