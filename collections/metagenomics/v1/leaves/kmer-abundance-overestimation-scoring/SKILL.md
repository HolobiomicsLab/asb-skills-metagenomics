---
name: kmer-abundance-overestimation-scoring
description: Use when comparing abundance-reporting AMQ data structures (e.g. fimpera-wrapped
  vs raw cBF) on how overestimated k-mer abundances trade off against filter size
  in bits.
when_to_use_negative:
- Abundance outputs were bucketed with log2 surjective functions (abundanceToIdentifierStrategy)
  instead of original (non-log) abundance storage — the Fig. 5 comparison assumes
  non-log storage
- Goal is presence/absence membership without abundance counts — fimpera's counting
  AMQ and this overestimation score target abundance queries
- Only underestimation or false-negative errors are of interest — this score counts
  exclusively overestimation errors (reported > original)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: fimpera
  role: Wraps a counting Bloom filter to index and query k-mers with abundances; its
    wrapped-query output is compared against raw cBF output using the overestimation
    score
  repo: https://github.com/lrobidou/fimpera
- name: Python
  role: Computes the sum-of-squared-error overestimation scores and plots the size-vs-score
    trade-off figure (fig5_sse_tradeoff.png)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:4ce697d9ef20170290750030d187b3c304b8add0b472e0b6d76ced354bf79686
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - Python
  - fimpera
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# kmer-abundance-overestimation-scoring

## Summary

Scores k-mer abundance queries by summing only overestimation errors (sum of squared differences between reported and original abundances) and compares this overestimation score against the space budget in bits for fimpera-wrapped versus raw counting Bloom filters, as in Fig. 5 of the fimpera paper.

## When to use

Use when comparing abundance-reporting AMQ data structures (e.g. fimpera-wrapped vs raw cBF) on how overestimated k-mer abundances trade off against filter size in bits.

## When NOT to use

- Abundance outputs were bucketed with log2 surjective functions (abundanceToIdentifierStrategy) instead of original (non-log) abundance storage — the Fig. 5 comparison assumes non-log storage
- Goal is presence/absence membership without abundance counts — fimpera's counting AMQ and this overestimation score target abundance queries
- Only underestimation or false-negative errors are of interest — this score counts exclusively overestimation errors (reported > original)

## Inputs

- Per-read average abundances from fimpera-wrapped queries (per-position abundance output per read, -1 for low-complexity k-mers)
- Per-read average abundances from raw counting Bloom filter (cBF) queries
- Original ground-truth abundances (e.g. KMC k-mer count file with one k-mer, one tab, and its abundance per line)
- Filter size settings in bits for each method

## Outputs

- Table of sum-of-squared-error overestimation scores vs filter size in bits for fimpera and cBF
- Size-vs-score trade-off figure saved as fig5_sse_tradeoff.png

## How to apply

Load per-read average abundances produced by fimpera-wrapped queries and by raw counting Bloom filter (cBF) queries, together with the original ground-truth abundances. For each filter size setting, accumulate squared differences between reported and original abundances, counting only overestimation errors (reported > original), to obtain the sum-of-squared-error overestimation score. Tabulate the score alongside the filter size in bits, using per-read average abundances with original (non-log) abundance storage — i.e. avoid the log2 abundance-bucketing strategies implemented in abundanceToIdentifierStrategy. Plot size-vs-score with fimpera and cBF as separate series and mark reference operating points: score 200000 at ~3.10x10^9 bits for fimpera (~3.87x10^8 slots of 8 bits) versus ~330.9x10^9 bits for raw cBF. Validation: fimpera must reach the same overestimation score at roughly two orders of magnitude (100x) fewer bits than the raw cBF.

## Related tools

- **fimpera** (Wraps a counting Bloom filter to index and query k-mers with abundances; its wrapped-query output is compared against raw cBF output using the overestimation score) — https://github.com/lrobidou/fimpera
- **Python** (Computes the sum-of-squared-error overestimation scores and plots the size-vs-score trade-off figure (fig5_sse_tradeoff.png))

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3; ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Recomputed overestimation score at the reference operating point equals 200000 for both methods
- At that score, fimpera's filter size is ~3.10x10^9 bits (~3.87x10^8 slots of 8 bits) while raw cBF is ~330.9x10^9 bits, a ~100x ratio
- Only squared differences where reported abundance exceeds the original are accumulated; underestimations contribute zero to the score
- Per-read average abundances were used with original (non-log) abundance storage, not log2-bucketed values
- Reproduced plot shows the same Fig. 5 trade-off curve ordering (fimpera strictly left of raw cBF) and is saved as fig5_sse_tradeoff.png

## Limitations

- k-mer abundances reported as -1 (low-complexity k-mers) must be handled/excluded before averaging per read
- Raw counting Bloom filters require ~two orders of magnitude more space than fimpera for the same overestimation score, so small-memory settings may be infeasible for the raw cBF baseline
- Abundance-to-identifier surjective functions (e.g. log2 bucketing) change reported abundance granularity; comparing across storage strategies invalidates the non-log Fig. 5 comparison
- The README notes index bit-per-bucket (-b) and z (s-mer size) parameters must be chosen; wrong settings shift the trade-off curve

## Evidence

- [abstract] Two-orders-of-magnitude space reduction at equal precision: "Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision."
- [abstract] Improved precision of reported abundances: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [readme] Index input format for ground-truth abundances: [evidence span withheld — anchor status: not_found]
- [readme] Low-complexity k-mers reported as -1: [evidence span withheld — anchor status: external_doc]
- [readme] Configurable abundance bucketing strategies: [evidence span withheld — anchor status: external_doc]
