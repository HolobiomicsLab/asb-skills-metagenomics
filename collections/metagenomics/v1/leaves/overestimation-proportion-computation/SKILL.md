---
name: overestimation-proportion-computation
description: 'Use when comparing abundance-call correctness of fimpera versus a same-size
  cAMQ/cBF: true-positive query results with reported abundances and ground truth
  are available and overestimation rates must be quantified.'
when_to_use_negative:
- Input is presence/absence query data without abundance values, since overestimation
  is defined relative to ground-truth abundances
- The indexes being compared are not identically sized — use the reduced-size (two-orders-of-magnitude
  smaller cBF) experiment instead
- Evaluating false positive calls rather than true-positive calls, since this skill
  classifies only called abundances against ground truth
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Produces the abundance query results (fimpera index, z=3) whose reported abundances
    are scored against ground truth alongside the original cBF results
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:21d04b4cc5d29e0de0c5f27d426cbeb28d60a230c632c68b3333088b9bb2a6e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - fimpera
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# overestimation-proportion-computation

## Summary

Compute correctness metrics for reported k-mer abundances — the proportion of overestimated abundance calls and the average distance of incorrect calls from ground truth — to quantify whether fimpera (z=3) improves abundance precision over querying an identically sized counting Bloom filter (cBF) directly.

## When to use

Use when comparing abundance-call correctness of fimpera versus a same-size cAMQ/cBF: true-positive query results with reported abundances and ground truth are available and overestimation rates must be quantified.

## When NOT to use

- Input is presence/absence query data without abundance values, since overestimation is defined relative to ground-truth abundances
- The indexes being compared are not identically sized — use the reduced-size (two-orders-of-magnitude smaller cBF) experiment instead
- Evaluating false positive calls rather than true-positive calls, since this skill classifies only called abundances against ground truth

## Inputs

- True-positive query results from an identically sized counting Bloom filter index (reported abundances paired with ground-truth abundances)
- True-positive query results from a fimpera (z=3) index (reported abundances paired with ground-truth abundances)
- KMC-style k-mer abundance file (one k-mer per line, tab, abundance) used to build the indexes

## Outputs

- metrics_table.csv containing per-method overestimation proportions (cBF 1.54%, fimpera 1.33%) and average incorrect-abundance distances (cBF 1.33, fimpera 1.07)

## How to apply

Load the true-positive query results (reported abundances vs ground truth) from two indexes of identical size: the original counting Bloom filter and the fimpera index with z=3 (s-mers of size k−3, e.g. k=35 yields 32-mers stored in the cBF). For each called k-mer abundance, classify it as exactly correct or overestimated relative to the ground-truth abundance. Compute the proportion of overestimated abundances per method — expected 1.54% for the original cBF versus 1.33% for fimpera (z=3). Then compute the average distance of incorrect (overestimated) abundances from the ground truth — expected 1.33 abundance ranges for cBF versus 1.07 for fimpera. Assemble both metrics into a single table (e.g. metrics_table.csv) so the improvement in precision of reported abundances is directly comparable across methods.

## Related tools

- **fimpera** (Produces the abundance query results (fimpera index, z=3) whose reported abundances are scored against ground truth alongside the original cBF results) — https://github.com/lrobidou/fimpera

## Examples

```
./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Recomputed overestimation proportions match the reported figures: 1.54% for the original cBF and 1.33% for fimpera (z=3)
- Recomputed average incorrect-abundance distances match: 1.33 abundance ranges for cBF and 1.07 for fimpera (z=3)
- fimpera's overestimation proportion is lower than the original cBF's, confirming improved precision of reported abundances
- Classification is exhaustive: every true-positive abundance call is labeled exactly correct or overestimated relative to ground truth
- Both indexes compared are of identical size (same-size cBF experiment)

## Limitations

- Metrics are computed only on true-positive calls, not on false-positive calls or all k-mers
- Results are specific to counting Bloom filter AMQs; fimpera is template-based and adaptable to other AMQs supporting abundance queries, but these figures were derived for cBF with z=3
- K-mers of too-low complexity may be reported as -1 and excluded from abundance correctness scoring

## Evidence

- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [results] On true-positive calls from the same-size cBF experiment, 1.54% of calls are overestimated with the original cBF versus 1.33% with fimpera: [evidence span withheld — anchor status: not_found]
- [readme] The abundance is displayed as -1 if the complexity of the kmer is too low.: [evidence span withheld — anchor status: external_doc]
