---
name: metrics-table-generation
description: Use when comparing abundance-call correctness of two counting-AMQ based
  indexes (e.g. fimpera z=3 vs original cBF of identical size) from true-positive
  query results with reported abundances and ground truth.
when_to_use_negative:
- The analysis goal is false-positive rate or index size/precision trade-off rather
  than abundance correctness on true-positive calls
- The query results lack ground-truth abundances, so overestimation and error distance
  cannot be classified
- The two indexes being compared are not identically sized, invalidating the same-size
  cBF comparison
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: fimpera
  role: Tool under evaluation; its z=3 index query results provide reported k-mer
    abundances to compare against cBF abundances on true-positive calls
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:f3f35b0d69f660bcfd8fba20e8eb2cb6c0d67635978aaf0fa3bd79b62218b6d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - fimpera
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# metrics-table-generation

## Summary

Reproduce abundance correctness metrics for fimpera versus a same-size counting Bloom filter (cBF): compute the proportion of overestimated k-mer abundances on true-positive calls and the average distance of incorrect abundances from ground truth, assembling both into a single metrics table (metrics_table.csv).

## When to use

Use when comparing abundance-call correctness of two counting-AMQ based indexes (e.g. fimpera z=3 vs original cBF of identical size) from true-positive query results with reported abundances and ground truth.

## When NOT to use

- The analysis goal is false-positive rate or index size/precision trade-off rather than abundance correctness on true-positive calls
- The query results lack ground-truth abundances, so overestimation and error distance cannot be classified
- The two indexes being compared are not identically sized, invalidating the same-size cBF comparison

## Inputs

- True-positive query results from the original same-size counting Bloom filter: reported abundances paired with ground-truth abundances per called k-mer
- True-positive query results from the fimpera (z=3) index: reported abundances paired with ground-truth abundances per called k-mer

## Outputs

- metrics_table.csv containing overestimation proportions and average incorrect-abundance distances for each method

## How to apply

Load the true-positive query results (reported k-mer abundances vs ground truth) produced by querying both a same-size counting Bloom filter and a fimpera index built with z=3. For each called k-mer abundance, classify it as exactly correct or overestimated relative to the ground-truth abundance, then compute the proportion of overestimated abundances per method. Next, restrict to incorrect (overestimated) calls and compute the average distance, in abundance ranges, from the ground truth for each method. Assemble the proportions and average distances into a single metrics table written to metrics_table.csv. The rationale is that fimpera improves the precision of reported abundances beyond just reducing false positives, so abundance correctness must be quantified separately from FP rate.

## Related tools

- **fimpera** (Tool under evaluation; its z=3 index query results provide reported k-mer abundances to compare against cBF abundances on true-positive calls) — https://github.com/lrobidou/fimpera

## Examples

```
./bin/fimpera_query index.idx query.fasta  # then classify reported vs ground-truth abundances into exact/overestimated and write metrics_table.csv
```

## Evaluation signals

- Overestimation proportion for the original cBF on true-positive calls equals 1.54%
- Overestimation proportion for fimpera (z=3) on true-positive calls equals 1.33%
- Average distance of incorrect (overestimated) abundances from ground truth equals 1.33 abundance ranges for cBF
- Average distance of incorrect (overestimated) abundances from ground truth equals 1.07 abundance ranges for fimpera (z=3)
- metrics_table.csv contains both metrics for both methods and fimpera values are lower than cBF values on both metrics

## Limitations

- Metrics are only meaningful on true-positive calls from identically sized indexes; different index sizes measure a different property (precision at reduced size)
- Distance is measured in abundance ranges rather than raw abundance units, so the metric depends on how abundance ranges are defined
- Low-complexity k-mers can be reported as -1 by fimpera and are excluded from abundance correctness evaluation

## Evidence

- [abstract] fimpera reduces the false positive rate of counting Bloom filters by two orders of magnitude and improves precision of reported abundances: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [intro] fimpera is a strategy for reducing false positive calls from AMQ structures with abundance queries: "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
- [readme] The fimpera implementation uses a counting Bloom filter and the z parameter controls s-mer size: "The `fimpera` implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences"
- [readme] Low-complexity kmers are reported with abundance -1 and a toy query run shows per-position reported abundances: [evidence span withheld — anchor status: external_doc]
