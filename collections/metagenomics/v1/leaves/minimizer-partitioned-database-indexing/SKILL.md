---
name: minimizer-partitioned-database-indexing
description: Use when implementing or reconstructing an exact k-mer lookup database
  (e.g., k-mer→LCA taxon maps) that must support fast per-k-mer queries without alignment,
  via minimizer grouping and indexed binary search.
when_to_use_negative:
- The task requires inexact alignment or sensitivity to mutations/indels rather than
  exact k-mer matches (Kraken's speed derives from exact-match database queries, not
  alignment).
- Query k-mers contain ambiguous nucleotides — Kraken ignores k-mers with ambiguous
  nucleotides, so they yield no lookup.
- A ranked/quantitative abundance estimate is needed from only a small fraction of
  reads; this scheme classifies every read via k-mer LCA voting rather than estimating
  abundance.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Implements the minimizer-partitioned database layout, index, and binary-search
    query procedure for k-mer LCA lookup during classification
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter whose database file format supplies the 4-byte
    count spaces reused to store k-mer LCA taxonomic IDs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:5cfd5b025eb307a910b5cad1e62dbb40f2a8f2f050ee458fe528cc4b95f0d78a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# minimizer-partitioned-database-indexing

## Summary

A database layout and lookup scheme (used by Kraken) that partitions k-mer/LCA records by their minimizer, stores each minimizer group consecutively sorted by canonical representation, and answers exact k-mer queries via a fixed-size minimizer index plus binary search within the group range.

## When to use

Use when implementing or reconstructing an exact k-mer lookup database (e.g., k-mer→LCA taxon maps) that must support fast per-k-mer queries without alignment, via minimizer grouping and indexed binary search.

## When NOT to use

- The task requires inexact alignment or sensitivity to mutations/indels rather than exact k-mer matches (Kraken's speed derives from exact-match database queries, not alignment).
- Query k-mers contain ambiguous nucleotides — Kraken ignores k-mers with ambiguous nucleotides, so they yield no lookup.
- A ranked/quantitative abundance estimate is needed from only a small fraction of reads; this scheme classifies every read via k-mer LCA voting rather than estimating abundance.

## Inputs

- k-mer/LCA database file (Jellyfish-format hash whose 4-byte count slots are overwritten with LCA taxonomic IDs)
- reference genome library sequences with NCBI taxonomy IDs (used upstream to set k-mer LCA values)
- query nucleotide sequences (e.g., FASTA/FASTQ reads) to classify

## Outputs

- minimizer-partitioned database file with minimizer-grouped, canonically sorted k-mer records
- fixed-size minimizer index file mapping each 15-bp minimizer value to offset and length of its group range
- per-k-mer LCA taxon IDs retrieved for query k-mers

## How to apply

Load the k-mer LCA database in which Jellyfish's 4-byte count spaces hold each k-mer's LCA taxonomic ID. For each k-mer, compute the canonical representation of its 15-bp (M=15) minimizer, first XOR-toggling half the bits of each M-mer's canonical representation before lexicographical comparison so the minimizer distribution is evened out. Store k-mers sharing a minimizer consecutively, sorted within each group by canonical representation, and build a fixed-size index mapping every possible 15-bp minimizer value to the offset and length of its group in the database file. To query a k-mer R, look up the index positions where k-mers with R's minimizer would be stored, then perform a binary search within that range to retrieve the LCA taxon ID. Cache the previous query's range and skip minimizer recomputation when consecutive queries share a minimizer, since reads are scanned k-mer by k-mer.

## Related tools

- **Kraken** (Implements the minimizer-partitioned database layout, index, and binary-search query procedure for k-mer LCA lookup during classification) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter whose database file format supplies the 4-byte count spaces reused to store k-mer LCA taxonomic IDs)

## Evaluation signals

- Query a sample of database k-mers and confirm the returned LCA taxon IDs match the source LCA database (output_matches_reference).
- Index file exists with the expected fixed-size structure covering every possible 15-bp minimizer value (file_exists, field_present on offset/length records).
- Within each minimizer group, k-mer records are sorted by canonical representation and all members share the group's minimizer (ordering invariant).
- Cached-range optimization triggers: consecutive queries with an unchanged minimizer are resolved by searching the previous range without recomputing the minimizer.
- End-to-end sanity check: k-mer lookups feeding the classification tree produce the expected root-to-leaf path scores / taxon labels for known reads.

## Limitations

- Only exact k-mer matches are supported; no inexact alignment, so reads from divergent or unsampled lineages may be misclassified at finer ranks (e.g., Pelosinus fermentans not identified at genus level).
- K-mers containing ambiguous nucleotides are ignored, so k-mers spanning 'N' characters do not affect classification (relevant when paired mates are concatenated with 'NNNNN').
- The index is fixed-size for M=15 minimizers, binding the layout to that minimizer length; MiniKraken's 4-GB downsampling (removing 18 of every 19 records) trades sensitivity for memory.

## Evidence

- [discussion] A query for a k-mer R can then be processed by looking up in an index the positions in the database where the k-mers with R's minimizer would be stored, and then performing a binary search within: [evidence span withheld — anchor status: not_found]
- [discussion] we use the exclusive-or (XOR) operation to toggle half of the bits of each M-mer's canonical representation prior to comparing the M-mers: [evidence span withheld — anchor status: not_found]
- [discussion] Rather than compute the minimizer each time we perform a query, we first search the previous range: "Rather than compute the minimizer each time we perform a query, we first search the previous range"
- [discussion] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers of the k-mers' LCA values: [evidence span withheld — anchor status: not_found]
- [intro] This speed advantage derives in large part from the use of exact-match database queries of k-mers, rather than inexact alignment of sequences: "This speed advantage derives in large part from the use of exact-match database queries of k-mers, rather than inexact alignment of sequences"
