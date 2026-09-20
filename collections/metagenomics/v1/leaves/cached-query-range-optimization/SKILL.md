---
name: cached-query-range-optimization
description: Use when reimplementing or tuning Kraken-style exact k-mer/LCA lookup
  over a minimizer-partitioned database and you classify consecutive k-mers of a read,
  so the prior query's range can be searched before recomputing the M=15 minimizer.
when_to_use_negative:
- Random-access queries of isolated k-mers (e.g., database validation sampling) where
  consecutive k-mers rarely share a minimizer — the cached range gives no benefit
  and a fresh index lookup is simpler
- A k-mer store that is not partitioned by minimizer with an offset/length index (e.g.,
  a plain sorted k-mer table), since the cached range would not bound valid records
  for the query
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: Kraken
  role: Implements the minimizer-grouped k-mer/LCA database, the index lookup plus
    binary-search query procedure, and the cached previous-range search with skipped
    minimizer recomputation
  repo: github.com/DerrickWood/kraken
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:4ac53598cfa1161b0639465f7a84fd2ec565e4af82ab0a7f873aff8779ee343e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cached-query-range-optimization

## Summary

Kraken's optimization for its minimizer-indexed k-mer/LCA database: after answering a k-mer query, the previous database range is cached, and subsequent queries first search that range and skip minimizer recomputation when the 15-bp minimizer is unchanged. This exploits the locality of consecutive k-mers within one read to avoid repeated index lookups.

## When to use

Use when reimplementing or tuning Kraken-style exact k-mer/LCA lookup over a minimizer-partitioned database and you classify consecutive k-mers of a read, so the prior query's range can be searched before recomputing the M=15 minimizer.

## When NOT to use

- Random-access queries of isolated k-mers (e.g., database validation sampling) where consecutive k-mers rarely share a minimizer — the cached range gives no benefit and a fresh index lookup is simpler
- A k-mer store that is not partitioned by minimizer with an offset/length index (e.g., a plain sorted k-mer table), since the cached range would not bound valid records for the query

## Inputs

- Kraken k-mer/LCA database (Jellyfish-style 4-byte count spaces overwritten with LCA taxonomic IDs), with k-mer records grouped by minimizer and sorted by canonical representation
- Fixed-size minimizer index mapping every possible 15-bp minimizer value to the offset and length of its record range
- Query DNA read decomposed into k-mers for classification

## Outputs

- LCA taxonomic ID for each queried k-mer retrieved via binary search
- Cached (offset, length) range from the previous query, reused when the minimizer is unchanged

## How to apply

First build the database so k-mers sharing a minimizer are stored consecutively, sorted by canonical representation, with a fixed-size index mapping every 15-bp minimizer value to the offset and length of its group's range. A query for k-mer R is normally processed by looking up the index positions where k-mers with R's minimizer would be stored, then binary-searching within that range to retrieve the LCA taxon ID. The cached-range optimization replaces the index-lookup step for consecutive queries: rather than compute the minimizer each time, first binary-search the previous query's range, and only if the k-mer is absent from that range recompute the canonical 15-bp minimizer (applying XOR to toggle half the bits of each M-mer's canonical representation before lexicographical comparison) and perform a fresh index lookup. The rationale is that neighboring k-mers in a read frequently share the same minimizer, so the recomputation and index lookup can be skipped for most of a read's k-mers.

## Related tools

- **Kraken** (Implements the minimizer-grouped k-mer/LCA database, the index lookup plus binary-search query procedure, and the cached previous-range search with skipped minimizer recomputation) — github.com/DerrickWood/kraken

## Evaluation signals

- Query a sample of database k-mers and confirm returned LCA taxon IDs match the source LCA database (output_matches_reference on the verification report)
- Verify the index file exists and every record carries offset and length fields for its 15-bp minimizer group (file_exists, field_present)
- Instrument the query loop: for consecutive k-mers of one read, the number of minimizer recomputations and index lookups should be far smaller than the number of k-mer queries
- Cross-check: for k-mers whose minimizer matches the previous query's, binary search of the cached range must return the same LCA taxon ID as a full index lookup plus binary search
- A k-mer absent from the cached range must fall through to minimizer recomputation and a new index lookup, never return a wrong or empty LCA

## Limitations

- The cached range is only valid while the minimizer is unchanged; a new k-mer with a different minimizer forces full recomputation of the M=15 minimizer and a fresh index lookup
- The XOR toggling of half the bits of each M-mer's canonical representation exists to even out the minimizer distribution; skewed minimizers would reduce the locality the cache relies on
- Some structural details of the search (e.g., Figure 5's database-search illustration) are referenced but not included in the article text, so exact index sizing must be inferred from the workflow description

## Evidence

- [discussion] cached-range optimization rationale: "Rather than compute the minimizer each time we perform a query, we first search the previous range"
- [discussion] index lookup plus binary search query procedure: [evidence span withheld — anchor status: not_found]
- [discussion] XOR toggling for even minimizer distribution: [evidence span withheld — anchor status: not_found]
