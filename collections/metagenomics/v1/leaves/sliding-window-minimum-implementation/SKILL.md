---
name: sliding-window-minimum-implementation
description: Use when computing minima over all fixed-size sliding windows of a numeric
  vector in linear time, especially in C++ where auxiliary allocations must be avoided
  and an in-place minL/minR scheme is preferred.
when_to_use_negative:
- When dynamic memory allocation is acceptable and a deque-based sliding window minimum
  would be simpler
- When windows are variable-sized or overlapping patterns other than fixed stride-1
  sliding windows are needed
edam_operation: http://edamontology.org/operation_2409
tools:
- name: sliding-minimum-windows
  role: Reference implementation benchmarking naive, reference-min, fixed window (out-of-place
    and in-place), deque, deque in-place, and rotation deque sliding minimum algorithms;
    compiled via g++ main.cpp min.cpp -o sliding -O3
  repo: https://github.com/lrobidou/sliding-minimum-windows
- name: fimpera
  role: Source article whose k-mer abundance querying motivated the sliding window
    minimum work by the same authors
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:df2cec1365c1bf37469814129c10b2304a4b52f9a2b2a95a951511e0df565b75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/sliding-minimum-windows
  tools:
  - fimpera
  - sliding-minimum-windows
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sliding-window-minimum-implementation

## Summary

Implements and benchmarks an in-place fixed-window sliding-minimum algorithm that splits an input vector into fixed non-overlapping windows of size W, computes per-window prefix (minL) and suffix (minR) minima in place, and yields each sliding window's minimum without dynamic memory allocation.

## When to use

Use when computing minima over all fixed-size sliding windows of a numeric vector in linear time, especially in C++ where auxiliary allocations must be avoided and an in-place minL/minR scheme is preferred.

## When NOT to use

- When dynamic memory allocation is acceptable and a deque-based sliding window minimum would be simpler
- When windows are variable-sized or overlapping patterns other than fixed stride-1 sliding windows are needed

## Inputs

- Input vector of numeric values over which sliding window minima are computed
- Fixed window size W (edge cases include W=1 and W=|v|)
- Test arrays covering repeated minima and boundary window sizes

## Outputs

- Verified minL/minR correctness outputs for all test arrays (plain-text results file)
- sliding_minimum_benchmark.csv with columns for array length, window size, and measured runtimes of in-place and naive implementations
- In-place minL and minR vectors encoding every sliding window minimum

## How to apply

Retrieve the reference implementation at github.com/lrobidou/sliding-minimum-windows to mirror its interface and algorithmic contract. Split the input vector of values into fixed, non-overlapping windows of size W, then compute per-window prefix minima (minL) and suffix minima (minR) in place, combining them so that each sliding window's minimum is produced without auxiliary allocations. Validate correctness by testing edge cases (W=1, W=|v|, arrays with repeated minima) and checking that every minL/minR-derived value equals the brute-force minimum over the corresponding window. Benchmark the in-place implementation against a naive O(W*|v|) baseline that recomputes the minimum per window, over increasing array sizes, writing timings to sliding_minimum_benchmark.csv with columns for array length, window size, and both runtimes. Write verified minL/minR correctness outputs for all test arrays to a plain-text results file.

## Related tools

- **sliding-minimum-windows** (Reference implementation benchmarking naive, reference-min, fixed window (out-of-place and in-place), deque, deque in-place, and rotation deque sliding minimum algorithms; compiled via g++ main.cpp min.cpp -o sliding -O3) — https://github.com/lrobidou/sliding-minimum-windows
- **fimpera** (Source article whose k-mer abundance querying motivated the sliding window minimum work by the same authors) — https://github.com/lrobidou/fimpera

## Examples

```
git clone https://github.com/lrobidou/sliding-minimum-windows; cd sliding-minimum-windows; g++ main.cpp min.cpp -o sliding -O3; ./sliding
```

## Evaluation signals

- Every minL/minR-derived window minimum equals the brute-force minimum over the corresponding window for all test arrays
- Edge cases pass: W=1, W=|v|, and arrays with repeated minima produce correct outputs
- In-place implementation completes with no auxiliary/dynamic allocations for the output
- Benchmark timings show the in-place fixed-window variant against the naive O(W*|v|) baseline in sliding_minimum_benchmark.csv over increasing array lengths

## Limitations

- The fixed-window scheme targets fixed, non-overlapping windows of size W; other windowing patterns are out of scope
- Correctness must be checked against a naive per-window recomputation baseline, which is O(W*|v|) and slow for large inputs

## Evidence

- [other] The proposed 'fixed window' algorithm computes sliding window minima by splitting the input vector of values into fixed, non-overlapping windows of size W: [evidence span withheld — anchor status: not_found]
- [other] on which per-window prefix/suffix minima (minL and minR) are then computed to yield each window's minimum in place: [evidence span withheld — anchor status: not_found]
- [readme] our proposal: the fixed window approach in place, as described in the paper: [evidence span withheld — anchor status: external_doc]
- [readme] g++ main.cpp min.cpp -o sliding -O3: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark of algorithms for finding minimum in sliding windows: [evidence span withheld — anchor status: external_doc]
