---
name: minimizer-index-database-query
description: Use when a metagenomic classifier must query each query-sequence k-mer
  against a large exact-match k-mer database (Jellyfish count spaces storing LCA taxon
  IDs) and per-k-mer alignment is too slow.
when_to_use_negative:
- When inexact/alignments-based similarity is required — Kraken's speed derives from
  exact-match database queries of k-mers rather than inexact alignment
- When query k-mers contain ambiguous nucleotides — Kraken ignores these k-mers, so
  they cannot be resolved by this query path
- When a sequence has no k-mers present in the database — it will simply be left unclassified
  rather than classified by homology
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: Kraken
  role: Classification engine whose database search performs the minimizer-index lookup
    plus binary search for each query k-mer and reuses the previous query range when
    the minimizer is unchanged
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to create the database; its 4-byte count
    spaces are overwritten to store k-mer LCA taxon IDs
- name: NCBI taxonomy database
  role: Source of taxon information used to compute LCA values stored for each k-mer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:f46503e07ce0c825a69573692e49f8096285ae9ef8717ea460821a4cd5a1412a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - NCBI taxonomy database
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# minimizer-index-database-query

## Summary

A method for rapidly resolving exact k-mer lookups in a Kraken-style database by grouping k-mers under their minimizer, storing them consecutively sorted by canonical representation, and answering each query with an index lookup followed by a binary search to recover the k-mer's LCA taxon ID.

## When to use

Use when a metagenomic classifier must query each query-sequence k-mer against a large exact-match k-mer database (Jellyfish count spaces storing LCA taxon IDs) and per-k-mer alignment is too slow.

## When NOT to use

- When inexact/alignments-based similarity is required — Kraken's speed derives from exact-match database queries of k-mers rather than inexact alignment
- When query k-mers contain ambiguous nucleotides — Kraken ignores these k-mers, so they cannot be resolved by this query path
- When a sequence has no k-mers present in the database — it will simply be left unclassified rather than classified by homology

## Inputs

- Kraken k-mer database (Jellyfish count spaces overwritten with k-mer LCA taxonomic IDs)
- Minimizer index mapping M-mer minimizers to database positions
- Query DNA sequences (metagenomic reads/contigs) with NCBI taxonomy tree available

## Outputs

- Per-k-mer LCA taxon ID retrieved from the database
- Set of k-mer LCA taxa feeding the weighted classification tree used to assign a taxonomic label per sequence (unclassified if no k-mer hits)

## How to apply

Build the database by counting k-mers with Jellyfish, then overwrite the 4-byte count spaces with the LCA taxonomic IDs of each k-mer, computing the LCA of stored and new taxa on conflicts while processing library sequences one at a time. Group the k-mers by their M-mer minimizer, store each group consecutively sorted by canonical representation, and build an index mapping minimizers to database positions. To even out minimizer distribution, XOR-toggle half the bits of each M-mer's canonical representation before lexicographical comparison. For each query k-mer R, look up in the index the positions where k-mers with R's minimizer are stored, then perform a binary search within that range to retrieve R's LCA taxon ID; skip ambiguous-nucleotide k-mers, and reuse the previous query range when the minimizer is unchanged rather than recomputing it.

## Related tools

- **Kraken** (Classification engine whose database search performs the minimizer-index lookup plus binary search for each query k-mer and reuses the previous query range when the minimizer is unchanged) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to create the database; its 4-byte count spaces are overwritten to store k-mer LCA taxon IDs)
- **NCBI taxonomy database** (Source of taxon information used to compute LCA values stored for each k-mer)

## Evaluation signals

- Every resolved query k-mer returns the correct LCA taxon ID via binary search within the index-provided range (verified by spot-checking stored values for library-derived k-mers)
- K-mers spanning ambiguous 'N' characters are skipped and do not affect classification
- When the minimizer is unchanged between consecutive k-mers, the previous query range is searched without recomputing the minimizer (no redundant index lookups)
- Sequences with no k-mer hits in the database are written to the classification output marked as unclassified
- Minimizer distribution is balanced after XOR-toggling half the bits of each M-mer's canonical representation (no single minimizer dominating index buckets)

## Limitations

- Relies on exact k-mer matches only; accuracy depends on the very large and still-growing number of sequenced microbial genomes (over 8,500) — k-mers absent from the database yield no evidence
- K-mers with ambiguous nucleotides are ignored, so k-mers spanning 'N' characters (e.g. from concatenated paired mates joined with 'NNNNN') do not affect classification
- Classification ties in the RTL path scoring are resolved by taking the LCA of the tied leaves, which can coarsen labels

## Evidence

- [discussion] A query for a k-mer R can then be processed by looking up in an index the positions in the database where the k-mers with R's minimizer would be stored, and then performing a binary search within: [evidence span withheld — anchor status: not_found]
- [discussion] we use the exclusive-or (XOR) operation to toggle half of the bits of each M-mer's canonical representation prior to comparing the M-mers: [evidence span withheld — anchor status: not_found]
- [discussion] Rather than compute the minimizer each time we perform a query, we first search the previous range: "Rather than compute the minimizer each time we perform a query, we first search the previous range"
- [discussion] Kraken ignores k-mers with ambiguous nucleotides, so the k-mers that span these 'N' characters do not affect classification: [evidence span withheld — anchor status: not_found]
- [readme] Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken.: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
