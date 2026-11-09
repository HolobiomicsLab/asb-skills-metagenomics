---
name: quadratic-baseline-runtime-benchmarking
description: Use when you have implemented an optimized sliding-window-minimum variant
  (e.g. fixed-window minL/minR in place) and need runtime comparisons against the
  naive per-window recomputation baseline over growing input arrays.
when_to_use_negative:
- When the goal is correctness verification of window minima rather than runtime scaling
  — use brute-force minL/minR comparison instead
- When the input requires dynamic-size or overlapping windows rather than fixed non-overlapping
  windows of size W
- When dynamic memory allocation is permitted and simpler deque-based solutions already
  meet performance needs
tools:
- name: sliding-minimum-windows
  role: Reference benchmark suite of sliding-window minimum algorithms (naive, reference-tracking,
    fixed-window not in place, fixed-window in place, deque variants) used to time
    and compare implementations
  repo: https://github.com/lrobidou/sliding-minimum-windows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:f9badab5151a720e7bfe2d595f299d331cc0e0b966a40cab7a716f401315c034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/sliding-minimum-windows
  tools:
  - sliding-minimum-windows
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# quadratic-baseline-runtime-benchmarking

## Summary

Benchmarks an optimized sliding-window-minimum implementation against the naive O(W*|v|) baseline that recomputes the minimum per window, over increasing array sizes, to quantify the runtime gain of the fixed-window prefix/suffix minima (minL/minR) approach.

## When to use

Use when you have implemented an optimized sliding-window-minimum variant (e.g. fixed-window minL/minR in place) and need runtime comparisons against the naive per-window recomputation baseline over growing input arrays.

## When NOT to use

- When the goal is correctness verification of window minima rather than runtime scaling — use brute-force minL/minR comparison instead
- When the input requires dynamic-size or overlapping windows rather than fixed non-overlapping windows of size W
- When dynamic memory allocation is permitted and simpler deque-based solutions already meet performance needs

## Inputs

- Numeric input vectors (arrays of values) covering edge cases: W=1, W=|v|, arrays with repeated minima, and random/increasing/decreasing value distributions
- Window size W parameter
- Compiled C++ benchmark binaries for the naive and optimized implementations

## Outputs

- sliding_minimum_benchmark.csv with columns for array length, window size, and measured runtimes of both implementations
- Runtime benchmark JSON files (e.g. for random, increasing, and decreasing input vectors)
- Plotted runtime-vs-array-size comparison graphs

## How to apply

Compile both implementations at the same optimization level (the reference repo builds with `g++ main.cpp min.cpp -o sliding -O3`). Run the naive baseline, which recomputes the minimum of each window of size W from scratch, and the proposed in-place fixed-window algorithm, which maintains minL/minR arrays in place with no auxiliary allocations. Time both over arrays of increasing length (the reference benchmark exercises random, increasing, and decreasing input vectors, since input monotonicity affects the deque and reference-tracking variants). Record array length, window size W, and both runtimes per measurement point — in the module workflow these timings are written to sliding_minimum_benchmark.csv — and plot the results to compare scaling behavior.

## Related tools

- **sliding-minimum-windows** (Reference benchmark suite of sliding-window minimum algorithms (naive, reference-tracking, fixed-window not in place, fixed-window in place, deque variants) used to time and compare implementations) — https://github.com/lrobidou/sliding-minimum-windows

## Examples

```
g++ main.cpp min.cpp -o sliding -O3 && ./sliding && python sliding.py
```

## Evaluation signals

- Benchmark CSV contains runtime rows for both the naive O(W*|v|) baseline and the in-place fixed-window implementation across multiple array lengths and window sizes
- The optimized implementation shows equal or better runtime scaling than the naive per-window recomputation as array size increases
- Timing is measured at identical optimization level (e.g. -O3) for both implementations so the comparison is fair
- Benchmark output covers three input regimes (random, increasing, decreasing vectors) as in the reference  outputs
- All tested arrays (W=1, W=|v|, repeated minima) pass the correctness check against brute-force window minima before timing

## Limitations

- Runtime conclusions depend on input distribution: monotonic (increasing/decreasing) vectors behave differently from random vectors for the deque-based and reference-tracking variants
- The in-place fixed-window algorithm's advantage holds only for fixed, non-overlapping windows of size W
- Timing comparisons require identical compilation flags; unoptimized builds can distort the baseline comparison

## Evidence

- [readme] Benchmark of algorithms for finding minimum in sliding windows: [evidence span withheld — anchor status: external_doc]
- [readme] algorithms tested list: [evidence span withheld — anchor status: external_doc]
- [readme] in-place fixed window proposal: [evidence span withheld — anchor status: external_doc]
- [readme] compile command: [evidence span withheld — anchor status: external_doc]
- [other] benchmark workflow: [evidence span withheld — anchor status: not_found]
- [readme] benchmark output files: [evidence span withheld — anchor status: not_found]
