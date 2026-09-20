---
name: abundance-error-distance-metrics
description: Use when comparing abundance accuracy of a counting AMQ (e.g. counting
  Bloom filter) against fimpera and you need overestimation proportions and error-distance
  statistics on true-positive calls.
when_to_use_negative:
- Input query results are false-positive calls rather than true-positive calls — this
  skill only characterizes abundance correctness on true positives
- The indexes being compared are not identically sized counting Bloom filters — size-reduction
  experiments (two-orders-of-magnitude smaller filters) measure a different property
  (precision at reduced size), not this same-size correctness comparison
- The AMQ does not support abundance queries, so reported abundances and overestimation
  distances cannot be computed
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: fimpera
  role: Provides the fimpera (z=3) index and query results whose reported abundances
    are compared against direct counting Bloom filter (cBF) query results
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Supplies the k-mer abundance input file (one k-mer per line with tab-separated
    abundance) required to build the indexes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:6666ae268e4c7f16852a0e8a06baf2075b761b2adb3ecfd9b36b26a19a91b71d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - fimpera
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# abundance-error-distance-metrics

## Summary

Reproduce the fimpera paper's abundance-correctness metrics by classifying true-positive k-mer abundance calls as exact or overestimated and computing the proportion of overestimations and the average distance of incorrect calls from ground truth, comparing an original cBF against fimpera (z=3) on identically sized indexes.

## When to use

Use when comparing abundance accuracy of a counting AMQ (e.g. counting Bloom filter) against fimpera and you need overestimation proportions and error-distance statistics on true-positive calls.

## When NOT to use

- Input query results are false-positive calls rather than true-positive calls — this skill only characterizes abundance correctness on true positives
- The indexes being compared are not identically sized counting Bloom filters — size-reduction experiments (two-orders-of-magnitude smaller filters) measure a different property (precision at reduced size), not this same-size correctness comparison
- The AMQ does not support abundance queries, so reported abundances and overestimation distances cannot be computed

## Inputs

- True-positive query results (reported abundance vs ground-truth abundance per k-mer) from the original same-size counting Bloom filter index
- True-positive query results (reported abundance vs ground-truth abundance per k-mer) from the fimpera (z=3) index of the same size
- KMC-format k-mer abundance file (one k-mer per line, tab-separated abundance) used to build the indexes

## Outputs

- metrics_table.csv containing per-method overestimation proportion (expected 1.54% cBF vs 1.33% fimpera) and average incorrect-abundance distance from ground truth (expected 1.33 cBF vs 1.07 fimpera)

## How to apply

Load reported k-mer abundances and ground-truth abundances for true-positive calls from two identically sized counting Bloom filter indexes: one queried directly and one queried through fimpera with z=3 (s-mers of size k-z). For each called k-mer abundance, classify it as exactly correct or overestimated relative to the ground-truth abundance, then compute the proportion of overestimated abundances per method. For the incorrect (overestimated) calls, compute the average abundance-range distance from the ground truth. Assemble both metrics into a single table (metrics_table.csv) and check against the expected figures: 1.54% overestimated with the original cBF vs 1.33% with fimpera, and average distances of 1.33 (cBF) vs 1.07 (fimpera). The rationale is that fimpera queries s-mers instead of k-mers, which improves the precision of reported abundances without changing the filter size.

## Related tools

- **fimpera** (Provides the fimpera (z=3) index and query results whose reported abundances are compared against direct counting Bloom filter (cBF) query results) — https://github.com/lrobidou/fimpera
- **KMC** (Supplies the k-mer abundance input file (one k-mer per line with tab-separated abundance) required to build the indexes)

## Examples

```
./bin/fimpera_index KMCfile.txt index.idx 1000000 -k 35 -b 5 -z 3; ./bin/fimpera_query index.idx query.fasta
```

## Evaluation signals

- Overestimation proportion recomputes to 1.54% for the original cBF and 1.33% for fimpera (z=3) on true-positive calls
- Average distance of incorrect (overestimated) abundances from ground truth recomputes to 1.33 for cBF and 1.07 for fimpera
- fimpera values are lower than cBF values on both metrics (less frequent and less severe overestimation), consistent with the paper's claim of improved abundance precision
- Both indexes compared are identically sized counting Bloom filters; classification of each call as exact vs overestimated is complete (no calls left unclassified)

## Limitations

- fimpera's reported abundances are bucketed abundance ranges, so distances are measured in abundance ranges rather than exact counts
- The same-size comparison only covers true-positive calls; separate experiments are needed for the false-positive rate reduction and the size-reduction (two-orders-of-magnitude smaller filter) claims
- Abundances may be displayed as -1 for k-mers whose complexity is too low, which must be excluded from abundance-correctness calculations

## Evidence

- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [results] 1.54% of calls are overestimated with the original cBF versus 1.33% with fimpera; among incorrect abundance calls, overestimations average 1.33 abundance ranges from the ground truth with cBF versus 1.07 with fimpera: [evidence span withheld — anchor status: not_found]
- [readme] The `fimpera` implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not).: "The `fimpera` implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences"
- [readme] The abundance is displayed as -1 if the complexity of the kmer is too low.: [evidence span withheld — anchor status: external_doc]
