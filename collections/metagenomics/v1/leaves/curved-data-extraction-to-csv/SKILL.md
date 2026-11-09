---
name: curved-data-extraction-to-csv
description: Use when a sweep of counting Bloom filter sizes has been indexed with
  fimpera over the TARA dataset and the raw FPR-vs-memory values (e.g. 2.98e9 bits
  at 2% FPR) must be tabulated into a CSV curve file for comparison with a no-fimpera
  baseline.
when_to_use_negative:
- The goal is query-time or abundance-precision evaluation of fimpera rather than
  the space-versus-FPR trade-off curve (no CSV curve table is the target output).
- Input is a single fixed-size index with no sweep of filter sizes and no matched
  baseline, so no FPR-vs-bits curve can be constructed.
- The AMQ under study does not support abundance queries — fimpera is defined only
  for AMQs supporting abundance queries, so no comparison curve can be produced.
tools:
- name: fimpera
  role: Builds fimpera-wrapped counting Bloom filter indexes over the filter-size
    sweep and queries them to measure false positive rates; its KMC count-file input
    format drives the indexing step.
  repo: https://github.com/lrobidou/fimpera
- name: counting Bloom filter
  role: Baseline AMQ built at identical filter sizes without fimpera to provide the
    ~65.5x10^9 bits-at-2%-FPR comparison points in the curve table.
- name: KMC
  role: Provides the k-mer/abundance count file consumed by fimpera_index (any k-mer
    counter outputting that format is accepted).
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:1008529a0f8239b99410e349db5072040315a3a51925bf6fff4d23a49b376ed5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - counting Bloom filter
  - fimpera
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# curved-data-extraction-to-csv

## Summary

Extracts the space-versus-false-positive-rate trade-off curve (Fig. 4 of the fimpera paper) into a structured CSV comparing fimpera-wrapped counting Bloom filters against a plain cBF baseline. This turns the paper's headline two-orders-of-magnitude memory claim into a machine-readable curve table for reuse and verification.

## When to use

Use when a sweep of counting Bloom filter sizes has been indexed with fimpera over the TARA dataset and the raw FPR-vs-memory values (e.g. 2.98e9 bits at 2% FPR) must be tabulated into a CSV curve file for comparison with a no-fimpera baseline.

## When NOT to use

- The goal is query-time or abundance-precision evaluation of fimpera rather than the space-versus-FPR trade-off curve (no CSV curve table is the target output).
- Input is a single fixed-size index with no sweep of filter sizes and no matched baseline, so no FPR-vs-bits curve can be constructed.
- The AMQ under study does not support abundance queries — fimpera is defined only for AMQs supporting abundance queries, so no comparison curve can be produced.

## Inputs

- TARA dataset sequencing reads (FASTQ/FASTA) for building k-mer abundance inputs
- KMC k-mer count file (one k-mer per line, tab-separated abundance) as required by fimpera_index
- Sweep of counting Bloom filter sizes with per-size fimpera and baseline cBF indexes
- Query files (FASTA/FASTQ, gzipped or not) used to measure false positive rates

## Outputs

- fig4_curve.csv with columns: filter size, false positive rate, memory in bits, and method (fimpera vs baseline)
- FPR-vs-bits curve table for both fimpera-wrapped and plain counting Bloom filters

## How to apply

Load the TARA dataset and build a sweep of fimpera indexes over counting Bloom filter sizes, indexing k-mers with their original (non-log) abundances via fimpera_index, using KMC k-mer count files (one k-mer per line, tab, abundance). Build matched plain counting Bloom filter (baseline, no fimpera) indexes at identical filter sizes. Query each index with fimpera_query to measure false positive rate and memory footprint in bits at every filter size. Extract the paper's Fig. 4 anchor points — at 2% FPR, ~2.98x10^9 bits with fimpera versus ~65.5x10^9 bits without; at 0.02% FPR, ~7.25x10^9 bits versus >360x10^9 bits — and write both methods' FPR-vs-bits values into fig4_curve.csv with columns for filter size, FPR, memory in bits, and method (fimpera vs baseline).

## Related tools

- **fimpera** (Builds fimpera-wrapped counting Bloom filter indexes over the filter-size sweep and queries them to measure false positive rates; its KMC count-file input format drives the indexing step.) — https://github.com/lrobidou/fimpera
- **counting Bloom filter** (Baseline AMQ built at identical filter sizes without fimpera to provide the ~65.5x10^9 bits-at-2%-FPR comparison points in the curve table.)
- **KMC** (Provides the k-mer/abundance count file consumed by fimpera_index (any k-mer counter outputting that format is accepted).)

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3 && ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- At 2% FPR, the fimpera row reports ~2.98x10^9 bits (~3.72x10^8 slots of 8 bits) while the baseline row reports ~65.5x10^9 bits — roughly 21-22x smaller.
- At 0.02% FPR, the fimpera row reports ~7.25x10^9 bits while the baseline exceeds 360x10^9 bits, consistent with a two-orders-of-magnitude size reduction at equal precision.
- fig4_curve.csv contains, for every filter size, paired fimpera and baseline rows sharing the same filter size and bits, differing only in the method column.
- Every emitted (filter size, FPR, bits) point matches the measured query outcome from fimpera_query on the corresponding index, with memory reported in bits (8-bit slots counted as their bit footprint).
- The fimpera curve is monotonically decreasing in FPR as filter size grows, and lies strictly below (fewer bits at equal FPR than) the baseline curve at all sizes.

## Limitations

- The fimpera implementation uses a counting Bloom filter; adapting the curve extraction to other AMQs requires re-implementing via fimpera's template mechanism.
- fimpera needs a k-mer/abundance file (one k-mer per line, one tab, abundance) — e.g. from KMC — so the pipeline depends on a separate k-mer counter step.
- K-mers whose complexity is too low are reported as abundance -1 and are not usable as normal query values.

## Evidence

- [abstract] fimpera reduces cBF size by two orders of magnitude at equal precision (Fig. 4 anchor): "Alternatively, fimpera allows for the reduction of the size of a counting Bloom filter by two orders of magnitude while maintaining the same precision."
- [abstract] Two-orders-of-magnitude FPR reduction motivating the FPR-vs-bits curve: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [readme] Index input format required before curve data can be generated: [evidence span withheld — anchor status: external_doc]
- [readme] Concrete index construction command used in the filter-size sweep: [evidence span withheld — anchor status: external_doc]
- [intro] Scope constraint for the comparison (AMQs supporting abundance queries): "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
