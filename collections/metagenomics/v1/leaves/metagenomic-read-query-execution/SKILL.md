---
name: metagenomic-read-query-execution
description: Use when benchmarking fimpera-style amplified s-mer querying of a counting
  Bloom filter on metagenomic FASTQ reads, specifically to verify that query time
  stays flat or slightly decreases as z increases.
when_to_use_negative:
- When the goal is assessing false positive rate or abundance precision rather than
  query execution time — use the FPR/precision benchmark instead.
- When the index is not a counting AMQ (e.g. no abundance support); fimpera's strategy
  requires an AMQ that supports abundance queries.
- When memory overhead of the amplified index is the quantity of interest — fimpera
  introduces no memory overhead, so timing benchmarks are uninformative there.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: fimpera
  role: Index k-mers with abundances into a counting Bloom filter and execute amplified
    s-mer queries of the read set at varying z to measure query time
  repo: https://github.com/lrobidou/fimpera
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:b3968e93b9671fd964bc268d0ed4d1b34eb90ace935ad34455da2867d599bb78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - fimpera
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metagenomic-read-query-execution

## Summary

Reproduce the fimpera query-time benchmark showing that wrapping a counting Bloom filter with fimpera does not increase query execution time as the z parameter grows; runtimes are tabulated against z alongside a raw-cBF baseline, mirroring Table 2 of the paper.

## When to use

Use when benchmarking fimpera-style amplified s-mer querying of a counting Bloom filter on metagenomic FASTQ reads, specifically to verify that query time stays flat or slightly decreases as z increases.

## When NOT to use

- When the goal is assessing false positive rate or abundance precision rather than query execution time — use the FPR/precision benchmark instead.
- When the index is not a counting AMQ (e.g. no abundance support); fimpera's strategy requires an AMQ that supports abundance queries.
- When memory overhead of the amplified index is the quantity of interest — fimpera introduces no memory overhead, so timing benchmarks are uninformative there.

## Inputs

- First 3,000,000 reads of TARA Oceans sample ERR4691696 in FASTQ format (query set)
- Pre-built counting Bloom filter index of TARA k-mers with abundances (fimpera index file, e.g. from a KMC k-mer count file)
- z parameter sweep values (number of extra s-mers per k-mer)

## Outputs

- Table of wall-clock query times vs z with the raw counting Bloom filter baseline (mirroring Table 2)
- Per-read per-position abundance calls emitted by fimpera_query

## How to apply

Build (or load a pre-built) counting Bloom filter index of TARA Oceans k-mers with fimpera_index, then query the first 3,000,000 reads of sample ERR4691696 (FASTQ) with fimpera_query. First query the raw counting Bloom filter directly (z=0 baseline) recording wall-clock time, then repeat through fimpera at successively increasing z values (each k-mer is split into z+1 s-mers, so s = k - z). Record query time at each z and tabulate time vs z next to the raw-cBF baseline, mirroring Table 2. Rationale: k-mers spanning an absent s-mer can be skipped during amplified queries, so larger z slightly reduces running time while fimpera adds no memory overhead.

## Related tools

- **fimpera** (Index k-mers with abundances into a counting Bloom filter and execute amplified s-mer queries of the read set at varying z to measure query time) — https://github.com/lrobidou/fimpera

## Examples

```
./bin/fimpera_query index.idx ERR4691696_3M.fastq
```

## Evaluation signals

- Query time at each z is equal to or slightly lower than the raw-cBF baseline (fimpera does not increase query time)
- Query time is monotonically non-increasing (or flat) as z increases, consistent with k-mers spanning absent s-mers being skipped
- fimpera adds no memory overhead relative to the raw counting Bloom filter across all z values tested
- fimpera_query runs successfully on the FASTQ read set and emits per-base abundance values (with -1 for low-complexity k-mers) without errors
- Index was built with matching k and z parameters (s = k - z) so the query exercises the intended amplified s-mer set

## Limitations

- Running-time benefit depends on k-mers spanning absent s-mers; query time behavior may differ on datasets where nearly all s-mers are present.
- Abundances are reported as -1 for k-mers whose complexity is too low, so some positions yield no abundance measurement.
- Indexing requires a k-mer abundance file (one k-mer, tab, abundance per line, e.g. from KMC); fimpera cannot index raw FASTA/FASTQ directly.

## Evidence

- [results] fimpera does not increase query execution time; running time slightly decreases as z increases: "fimpera does not introduce any memory overhead and may even reduces the query time."
- [readme] z parameter meaning and rationale: [evidence span withheld — anchor status: external_doc]
- [readme] fimpera query invocation pattern: "# querying a file
./bin/fimpera_query <index_name> <your query file>"
- [intro] fimpera applicability to abundance AMQs: "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
- [readme] indexing input format requirement: [evidence span withheld — anchor status: external_doc]
