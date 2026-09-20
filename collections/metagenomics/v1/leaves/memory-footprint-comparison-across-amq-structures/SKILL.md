---
name: memory-footprint-comparison-across-amq-structures
description: Use when evaluating how a k-mer abundance index (fimpera vs raw counting
  Bloom filter) trades memory footprint in bits against false positive rate, e.g.
  to size a filter for a 2% FPR target.
when_to_use_negative:
- The task is to improve FPR for an AMQ that does not support abundance queries (fimpera
  requires count-supporting AMQs)
- Only presence/absence membership matters and no false-positive-vs-memory curve is
  needed — a single index build suffices
- The index must store exact abundances with no approximation; fimpera's gain comes
  from coarsening (surjective abundance mapping) and cannot preserve exactness
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Wraps a counting Bloom filter to index k-mers with abundances and measure
    the FPR-versus-memory trade-off at each filter size in the sweep
  repo: https://github.com/lrobidou/fimpera
- name: counting Bloom filter
  role: Baseline cAMQ against which the fimpera-wrapped filter's memory footprint
    and FPR are compared at matched filter sizes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:a40b11f451c9735d99acc2f0fc4c064e25f2bcd29486cd8fedf5593cd6036dd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - counting Bloom filter
  - fimpera
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# memory-footprint-comparison-across-amq-structures

## Summary

Reproduces the space-versus-false-positive-rate trade-off of counting Approximate Membership Query data structures, comparing fimpera-wrapped counting Bloom filters against plain cBF baselines. Demonstrates that fimpera reaches a target FPR (e.g. 2%) with up to two orders of magnitude less memory.

## When to use

Use when evaluating how a k-mer abundance index (fimpera vs raw counting Bloom filter) trades memory footprint in bits against false positive rate, e.g. to size a filter for a 2% FPR target.

## When NOT to use

- The task is to improve FPR for an AMQ that does not support abundance queries (fimpera requires count-supporting AMQs)
- Only presence/absence membership matters and no false-positive-vs-memory curve is needed — a single index build suffices
- The index must store exact abundances with no approximation; fimpera's gain comes from coarsening (surjective abundance mapping) and cannot preserve exactness

## Inputs

- KMC-style abundance file (one k-mer per line, tab, abundance) built from the TARA dataset
- Sweep of counting Bloom filter sizes (number of buckets / bits)
- fimpera index parameters: k (e.g. 35), z (s-mer size, e.g. 3), bits per bucket (-b, e.g. 8 for the Fig. 4 comparison)

## Outputs

- fig4_curve.csv with columns: filter size, FPR, memory in bits, method (fimpera vs baseline)
- Per-index measured false positive rates and memory footprints in bits
- Comparison statement of space reduction (e.g. ~2.98x10^9 vs ~65.5x10^9 bits at 2% FPR)

## How to apply

Index the TARA dataset k-mers (with original, non-log abundances) using fimpera over a sweep of counting Bloom filter sizes (e.g. `./bin/fimpera_index <KMCfile.txt> index.idx -b <bits per bucket> -k 35 -z 3`), and build matched plain cBF baseline indexes at the same sizes. Query each index to measure the false positive rate and record memory in bits per filter size. Extract the paper's Fig. 4 anchor points: at 2% FPR, fimpera needs ~2.98x10^9 bits (3.72x10^8 slots of 8 bits) versus ~65.5x10^9 bits for the baseline (~21-22x smaller); at 0.02% FPR, ~7.25x10^9 bits versus >360x10^9 bits. Emit a CSV (fig4_curve.csv) with columns for filter size, FPR, memory in bits, and method (fimpera vs baseline), then validate that the fimpera curve sits 1-2 orders of magnitude below the baseline at equal precision.

## Related tools

- **fimpera** (Wraps a counting Bloom filter to index k-mers with abundances and measure the FPR-versus-memory trade-off at each filter size in the sweep) — https://github.com/lrobidou/fimpera
- **counting Bloom filter** (Baseline cAMQ against which the fimpera-wrapped filter's memory footprint and FPR are compared at matched filter sizes)

## Examples

```
./bin/fimpera_index kmc_abundances.txt index.idx <num_bits> -k 35 -b 8 -z 3; ./bin/fimpera_query index.idx query.fasta
```

## Evaluation signals

- At 2% FPR the fimpera curve reports ~2.98x10^9 bits (~3.72x10^8 slots of 8 bits), while the baseline reports ~65.5x10^9 bits (~21-22x more)
- At 0.02% FPR the fimpera curve reports ~7.25x10^9 bits while the baseline exceeds 360x10^9 bits
- The CSV contains both methods (fimpera and baseline) with matched filter sizes, and fimpera FPR is consistently 1-2 orders of magnitude lower at equal bits
- Memory overhead of fimpera over the raw cBF at matched filter size is zero (footprints identical for the same number of buckets)
- Indexes are built with non-log abundances; switching to log2-grouped abundances should be treated as a separate configuration

## Limitations

- fimpera implementation provided uses a counting Bloom filter; adapting other AMQ structures requires writing template instantiations and abundance-mapping classes
- fimpera indexing requires a pre-computed k-mer abundance file (e.g. KMC output), so the pipeline depends on an upstream counter tool
- Queries of low-complexity k-mers return abundance -1 rather than a count, which can affect measured query behavior
- The paper's reproduction sections were left as TODO in the README, so exact Fig. 4 reproduction scripts are not provided out of the box

## Evidence

- [abstract] fimpera reduces the false positive rate of counting Bloom filters by two orders of magnitude and improves precision of reported abundances: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [abstract] fimpera allows same precision with far less filter size: "Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision."
- [readme] fimpera indexing CLI parameters: [evidence span withheld — anchor status: external_doc]
- [readme] input format requirement for indexing: [evidence span withheld — anchor status: external_doc]
