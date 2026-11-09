---
name: algorithm-correctness-testing
description: Use when an in-place, allocation-free algorithm implementation (e.g.
  sliding-window minima over fixed windows of size W) must be proven correct against
  a brute-force oracle and benchmarked before adoption.
when_to_use_negative:
- When the analysis goal is k-mer abundance indexing or querying itself — that is
  fimpera's fimpera_index/fimpera_query pipeline, not the sliding-minimum test harness.
- When windows are variable-size or streaming and dynamic allocation of the output
  vector is acceptable — the deque or 'rotation deque' variants, not the fixed in-place
  window approach, are the right comparison points.
- When inputs are not numeric vectors whose per-window minimum is well-defined (the
  oracle check 'brute-force minimum over the corresponding window' has no meaning
  otherwise).
tools:
- name: sliding-minimum-windows
  role: Reference implementation and benchmark harness comparing the naive approach,
    last-min reference, fixed-window (in place and not in place), and deque-based
    sliding-minimum algorithms
- name: fimpera
  role: Host C++ template codebase whose no-memory-overhead design motivated the in-place
    (allocation-free) sliding-minimum module being validated
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:63d70e538dba99ba065d8120ee342f33d2367c5f85cf5d449a43fabdd4761079
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - fimpera
  - sliding-minimum-windows
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# algorithm-correctness-testing

## Summary

A validation-and-benchmark skill for the fixed-window sliding-minimum algorithm: it verifies that an in-place implementation's minL/minR outputs equal brute-force per-window minima and compares its runtime against the naive O(W*|v|) recompute-per-window baseline.

## When to use

Use when an in-place, allocation-free algorithm implementation (e.g. sliding-window minima over fixed windows of size W) must be proven correct against a brute-force oracle and benchmarked before adoption.

## When NOT to use

- When the analysis goal is k-mer abundance indexing or querying itself — that is fimpera's fimpera_index/fimpera_query pipeline, not the sliding-minimum test harness.
- When windows are variable-size or streaming and dynamic allocation of the output vector is acceptable — the deque or 'rotation deque' variants, not the fixed in-place window approach, are the right comparison points.
- When inputs are not numeric vectors whose per-window minimum is well-defined (the oracle check 'brute-force minimum over the corresponding window' has no meaning otherwise).

## Inputs

- Input vector of numeric values over which sliding-window minima are computed
- Fixed window size W (edge cases: W=1 and W=|v|)
- Reference implementation from the lrobidou/sliding-minimum-windows repository
- Naive O(W*|v|) baseline that recomputes the minimum for each window

## Outputs

- In-place minL/minR arrays equal to the brute-force per-window minima
- sliding_minimum_benchmark.csv with columns: array length, window size, runtimes of both implementations
- Plain-text results file with verified minL/minR correctness outputs for all test arrays
- JSON benchmark files (one per input distribution) written to the json folder

## How to apply

Retrieve the reference implementation at github.com/lrobidou/sliding-minimum-windows to mirror its interface and algorithmic contract, then implement the sliding window minimum as an independent C++ module that maintains minL/minR in place over fixed, non-overlapping windows of size W with no auxiliary dynamic allocations (the in-place constraint mirrors fimpera's claim of no memory overhead). Run the module on edge-case arrays covering W=1, W=|v|, and arrays with repeated minima, and assert that every minL/minR value equals the brute-force minimum over the corresponding window. Benchmark the in-place variant against the naive O(W*|v|) baseline that recomputes the minimum per window, compiling with 'g++ main.cpp min.cpp -o sliding -O3'. Write timings to sliding_minimum_benchmark.csv with columns for array length, window size, and measured runtimes of both implementations, and write verified minL/minR correctness outputs for all test arrays to a plain-text results file.

## Related tools

- **sliding-minimum-windows** (Reference implementation and benchmark harness comparing the naive approach, last-min reference, fixed-window (in place and not in place), and deque-based sliding-minimum algorithms)
- **fimpera** (Host C++ template codebase whose no-memory-overhead design motivated the in-place (allocation-free) sliding-minimum module being validated) — https://github.com/lrobidou/fimpera

## Examples

```
g++ main.cpp min.cpp -o sliding -O3 && ./sliding && python sliding.py
```

## Evaluation signals

- Every minL/minR value output by the module equals the brute-force minimum recomputed over the corresponding window for all test arrays, including W=1, W=|v|, and arrays with repeated minima.
- The in-place fixed-window variant produces outputs identical to the non-in-place fixed-window variant described in the paper.
- The in-place implementation performs no auxiliary dynamic memory allocations (output vector is preallocated).
- sliding_minimum_benchmark.csv exists with the required columns (array length, window size, runtimes of both implementations) and shows the in-place algorithm scaling better than the naive O(W*|v|) baseline as array size grows.
- The benchmark binary ./sliding produces 3 JSON files in the json folder covering random, increasing, and decreasing input vectors, and the plotted graphs appear in the figs folder.

## Limitations

- The fixed-window algorithm is validated only for fixed, non-overlapping windows of size W; deque-based sliding-minimum variants are separate implementations in the harness.
- Benchmark coverage is limited to three input distributions (random, increasing, decreasing vectors), so other data regimes are untested.
- Results depend on the build flags (the README specifies compiling with -O3), so timings may not transfer to other compilers or optimization levels.

## Evidence

- [readme] Benchmark of algorithms for finding minimum in sliding windows: [evidence span withheld — anchor status: external_doc]
- [readme] the fixed window approach in place, as described in the paper: [evidence span withheld — anchor status: external_doc]
- [readme] the naive one: recompute each window: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark outputs are 3 JSON files for random, increasing, and decreasing input vectors: [evidence span withheld — anchor status: external_doc]
- [abstract] fimpera introduces no memory overhead, motivating the in-place implementation: "fimpera does not introduce any memory overhead and may even reduces the query time."
