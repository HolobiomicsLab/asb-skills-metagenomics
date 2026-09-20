---
name: counting-bloom-filter-false-positive-analysis
description: Use when comparing an AMQ-with-counts index (fimpera) against a raw counting
  Bloom filter on abundance query accuracy, or when sizing a filter to hit a target
  overestimation score within a space budget.
when_to_use_negative:
- Abundances were stored as log2-bucketed values — the score computation assumes original
  (non-log) abundance storage
- The goal is presence/absence AMQ performance (classic false positive rate) rather
  than abundance-precision vs. space trade-off
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Indexes KMC k-mer abundances into a counting Bloom filter (parameters -k,
    -b bits per bucket, -z s-mer size) and performs the queries whose reported abundances
    feed the overestimation score
  repo: https://github.com/lrobidou/fimpera
- name: Python
  role: Computes the sum-of-squared-error overestimation scores from per-read abundance
    outputs and plots the size-vs-score trade-off figure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:cb3b9296900896a7084cffbec3568eebcf56429fb0b32ddd43c8989b16f14f26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - Python
  - fimpera
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# counting-bloom-filter-false-positive-analysis

## Summary

Benchmark workflow for quantifying the space-vs-accuracy trade-off of fimpera-wrapped counting Bloom filters versus raw cBFs for k-mer abundance queries, using a sum-of-squared-error overestimation score plotted against filter size in bits (e.g. Fig. 5 of the fimpera paper).

## When to use

Use when comparing an AMQ-with-counts index (fimpera) against a raw counting Bloom filter on abundance query accuracy, or when sizing a filter to hit a target overestimation score within a space budget.

## When NOT to use

- Abundances were stored as log2-bucketed values — the score computation assumes original (non-log) abundance storage
- The goal is presence/absence AMQ performance (classic false positive rate) rather than abundance-precision vs. space trade-off

## Inputs

- KMC-style k-mer abundance file (one k-mer per line, tab-separated abundance) for indexing
- Query read file (FASTQ or FASTA, gzipped or not)
- Ground-truth original abundances for the indexed k-mers
- Per-read average abundance outputs from fimpera-wrapped queries and raw cBF queries

## Outputs

- Tabulated overestimation (SSE) scores alongside filter size in bits for fimpera and raw cBF
- Size-vs-score trade-off plot saved as fig5_sse_tradeoff.png
- Per-read average abundance files per method and filter size

## How to apply

Index k-mer abundances (e.g. from a KMC output file listing one k-mer per line with its tab-separated abundance) into both a fimpera-wrapped counting Bloom filter and a raw cBF, sweeping filter sizes and optionally the z (s-mer) parameter. Query the same reads with both structures, storing per-read average abundances with original (non-log) abundance storage. For each filter size, compute the sum-of-squared-errors overestimation score by accumulating squared differences between reported and original abundances, counting only overestimation errors, then tabulate score against filter size in bits for both methods. Plot size-vs-score curves with fimpera and cBF as separate series and mark reference operating points, such as score 200000 at ~3.10×10^9 bits (fimpera, ~3.87×10^8 slots of 8 bits) versus ~330.9×10^9 bits for raw cBF — a ~100× space reduction at equal accuracy, consistent with the paper's two-orders-of-magnitude claim.

## Related tools

- **fimpera** (Indexes KMC k-mer abundances into a counting Bloom filter (parameters -k, -b bits per bucket, -z s-mer size) and performs the queries whose reported abundances feed the overestimation score) — https://github.com/lrobidou/fimpera
- **Python** (Computes the sum-of-squared-error overestimation scores from per-read abundance outputs and plots the size-vs-score trade-off figure)

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3 && ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Recomputed scores and bit sizes must reproduce the reported Fig. 5 trade-off: at overestimation score 200000, fimpera needs ~3.10×10^9 bits while raw cBF needs ~330.9×10^9 bits
- The cBF-to-fimpera bit ratio at equal score should be roughly 100× (two orders of magnitude), matching the paper's precision-preserved size-reduction claim
- Overestimation score must count only squared differences where the reported abundance exceeds the original abundance, excluding underestimation errors
- Filter size in bits must be consistent with the number of slots times bits per bucket (e.g. ~3.87×10^8 slots of 8 bits ≈ 3.10×10^9 bits for fimpera)

## Limitations

- fimpera's reported abundances can be imprecise per position; the toy example shows abundances of -1 for low-complexity k-mers, which must be handled when computing scores
- The fimpera implementation here uses a counting Bloom filter specifically; adapting to other AMQs requires reimplementing the template and the abundance-to-identifier surjective strategy
- Indexing requires a precomputed k-mer abundance file (e.g. from KMC); fimpera does not count k-mers itself

## Evidence

- [abstract] Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision.: "Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision."
- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision"
- [readme] fimpera needs a file containing the abundance of each k-mers to index them (each line should contain a k-mer (not a s-mer), one tab and the abundance associated with that k-mer).: [evidence span withheld — anchor status: not_found]
- [readme] The abundance is displayed as -1 if the complexity of the kmer is too low.: [evidence span withheld — anchor status: external_doc]
- [intro] fimpera relies on templates, hence it can be easily adapted to any other AMQ that supports abundance queries, for any usage.: "`fimpera` relies on templates, hence it can be easily adapted to any other AMQ that supports abundance queries, for any usage."
