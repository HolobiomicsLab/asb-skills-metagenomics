---
name: cbf-vs-fimpera-comparison-analysis
description: Use when comparing abundance accuracy of a counting Bloom filter queried
  directly versus via fimpera on identically sized cBF indexes, and you need overestimation
  rates and error-distance metrics.
when_to_use_negative:
- The indexes being compared are not the same size — use the size-reduction experiment
  (two orders of magnitude smaller fimpera index at equal precision) instead
- The analysis goal is false-positive call rate rather than abundance correctness
  on true-positive calls
- The AMQ does not support abundance queries (fimpera requires a counting AMQ)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Index k-mer abundances (with z=3, i.e. s-mers of size k-z in a counting Bloom
    filter) and answer abundance queries used for the correctness comparison against
    direct cBF queries
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:2884380328bd5a84bcf7ef4e99470825ebdddc26a74a7a4fd85165e8f6c493f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - fimpera
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cbf-vs-fimpera-comparison-analysis

## Summary

Reproduce the fimpera paper's abundance-correctness comparison: query identical-size counting Bloom filters directly versus through fimpera (z=3) and quantify overestimation proportions and error distances of reported k-mer abundances on true-positive calls.

## When to use

Use when comparing abundance accuracy of a counting Bloom filter queried directly versus via fimpera on identically sized cBF indexes, and you need overestimation rates and error-distance metrics.

## When NOT to use

- The indexes being compared are not the same size — use the size-reduction experiment (two orders of magnitude smaller fimpera index at equal precision) instead
- The analysis goal is false-positive call rate rather than abundance correctness on true-positive calls
- The AMQ does not support abundance queries (fimpera requires a counting AMQ)

## Inputs

- True-positive query results (reported abundance vs ground truth) from the identically sized counting Bloom filter index
- True-positive query results (reported abundance vs ground truth) from the fimpera index with z=3 on the same-size cBF

## Outputs

- Overestimation proportion per method (percent of abundance calls exceeding ground truth)
- Average distance from ground truth among incorrect (overestimated) abundance calls per method
- metrics_table.csv combining proportions and average error distances for cBF vs fimpera

## How to apply

Load query results (reported k-mer abundances vs ground truth) for true-positive calls from two same-size counting Bloom filter experiments: one queried directly and one queried through fimpera with z=3 (s-mers of size k-z indexed). For each called k-mer abundance, classify it as exactly correct or overestimated relative to the ground-truth abundance. Compute the proportion of overestimated abundances per method and the average distance of incorrect (overestimated) abundances from the ground truth. Assemble both metrics per method into a single table (metrics_table.csv). Correct reproduction yields overestimation proportions of 1.54% (direct cBF) vs 1.33% (fimpera, z=3) and average error distances of 1.33 (cBF) vs 1.07 (fimpera).

## Related tools

- **fimpera** (Index k-mer abundances (with z=3, i.e. s-mers of size k-z in a counting Bloom filter) and answer abundance queries used for the correctness comparison against direct cBF queries) — https://github.com/lrobidou/fimpera

## Examples

```
./bin/fimpera_index KMCfile.txt index.idx 1000000 -k 35 -b 5 -z 3 && ./bin/fimpera_query index.idx exemple_query.fasta
```

## Evaluation signals

- Overestimation proportion on true positives reproduces 1.54% for the direct cBF and 1.33% for fimpera (z=3)
- Average distance of incorrect abundance calls from ground truth reproduces 1.33 for cBF and 1.07 for fimpera (z=3)
- Both compared indexes have identical size (same-size cBF experiment)
- fimpera shows both a lower proportion of overestimations and a smaller average error distance than the direct cBF
- metrics_table.csv contains, for each method, both the overestimation proportion and the average incorrect-abundance distance

## Limitations

- fimpera indexes s-mers (size k-z) rather than full k-mers, so the reported abundance is derived from an average over s-mer counts rather than a direct k-mer count
- K-mers with too-low complexity are not queried and are reported as -1 by fimpera_query, so they must be excluded from correctness comparisons
- fimpera requires an external k-mer counting step (e.g. KMC) producing a tab-separated k-mer/abundance file before indexing
- Comparison is restricted to true-positive calls; false-positive rate analysis is a separate metric

## Evidence

- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [readme] fimpera is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries.: "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
- [readme] fimpera needs a file containing the abundance of each k-mers to index them (each line should contain a k-mer ... one tab and the abundance associated with that k-mer). KMC can provide such file: [evidence span withheld — anchor status: external_doc]
- [readme] The abundance is displayed as -1 if the complexity of the kmer is too low.: [evidence span withheld — anchor status: external_doc]
