---
name: kmer-abundance-query-evaluation
description: Use when you have reported k-mer abundances from one or more counting
  AMQ structures (e.g., counting Bloom filters, fimpera indices) plus ground-truth
  abundances and need to quantify abundance correctness on true-positive calls.
when_to_use_negative:
- You need raw false-positive presence/absence rates rather than correctness of reported
  abundance values on true-positive calls
- Comparing structures of different sizes — the reference experiment indexes identically
  sized counting Bloom filters, so size-vs-precision trade-offs are out of scope
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: 'Generates the abundance query results being evaluated: indexes k-mers (or
    s-mers, z=3) with abundance into a counting Bloom filter and reports per-k-mer
    abundances on query sequences'
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:a4387b0a37c036179c85f26ae73674082253e3605b869f3f80e18c7e4e371b53
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# kmer-abundance-query-evaluation

## Summary

Evaluates the correctness of k-mer abundance calls from counting AMQ data structures by comparing reported abundances against ground truth, quantifying the proportion of overestimated abundances and the average distance of incorrect calls from the true abundance (e.g., fimpera z=3 vs the original cBF).

## When to use

Use when you have reported k-mer abundances from one or more counting AMQ structures (e.g., counting Bloom filters, fimpera indices) plus ground-truth abundances and need to quantify abundance correctness on true-positive calls.

## When NOT to use

- You need raw false-positive presence/absence rates rather than correctness of reported abundance values on true-positive calls
- Comparing structures of different sizes — the reference experiment indexes identically sized counting Bloom filters, so size-vs-precision trade-offs are out of scope

## Inputs

- Reported k-mer abundances from a counting Bloom filter query (per k-mer, with ground truth)
- Reported k-mer abundances from a fimpera index (z=3) query (per k-mer, with ground truth)
- Ground-truth k-mer abundances (e.g., from a k-mer counter such as KMC output listing k-mer and abundance per line)

## Outputs

- Proportion of overestimated abundance calls per method (expected 1.54% cBF, 1.33% fimpera z=3)
- Average distance of incorrect (overestimated) abundances from ground truth per method (expected 1.33 cBF, 1.07 fimpera)
- Combined metrics table written to metrics_table.csv

## How to apply

Load the true-positive query results (reported abundances vs ground truth) from each structure — e.g., the identically sized counting Bloom filter and the fimpera index with z=3 (s-mers of size 32 for k=35). For each called k-mer abundance, classify it as exactly correct or overestimated relative to the ground-truth abundance, then compute the proportion of overestimated abundances per method. Separately, restrict to the incorrect (overestimated) calls and compute the average distance in abundance ranges from the ground truth. Assemble the proportions and average distances into a single metrics table (metrics_table.csv); in the reference experiment this reproduces 1.54% overestimated calls for the original cBF vs 1.33% for fimpera, and average overestimation distances of 1.33 vs 1.07, showing fimpera's reported abundances are more frequent and closer to the truth.

## Related tools

- **fimpera** (Generates the abundance query results being evaluated: indexes k-mers (or s-mers, z=3) with abundance into a counting Bloom filter and reports per-k-mer abundances on query sequences) — https://github.com/lrobidou/fimpera

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3; ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Overestimation proportion on true-positive calls reproduces 1.54% for the original cBF and 1.33% for fimpera (z=3)
- Average distance of incorrect abundance calls from ground truth reproduces 1.33 for cBF and 1.07 for fimpera (z=3)
- Every classified call is either exactly correct or overestimated relative to ground truth (no unclassified categories)
- metrics_table.csv contains both metric types (proportion and average distance) for every evaluated method

## Limitations

- Analysis is restricted to true-positive calls; it does not characterize false-positive k-mer calls, which fimpera reduces by two orders of magnitude but which are quantified elsewhere
- Metrics are specific to identically sized counting Bloom filter indices with z=3; other z values or index sizes require re-running the comparison
- fimpera reports -1 for low-complexity k-mers, which must be excluded or handled before abundance comparison

## Evidence

- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [readme] fimpera needs a file containing the abundance of each k-mers to index them (each line should contain a k-mer (not a s-mer), one tab and the abundance associated with that k-mer). KMC can provide such file.: [evidence span withheld — anchor status: external_doc]
- [readme] The abundance is displayed as -1 if the complexity of the kmer is too low.: [evidence span withheld — anchor status: external_doc]
- [readme] fimpera is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries.: "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
