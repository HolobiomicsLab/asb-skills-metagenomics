---
name: approximate-membership-query-benchmarking
description: Use when comparing an Approximate Membership Query data structure (e.g.,
  a counting Bloom filter) against its fimpera-wrapped version on abundance queries,
  to quantify false-positive rate, overestimation score, space, and query time.
when_to_use_negative:
- The data structure is a plain AMQ without counts (no abundance support), since fimpera
  requires abundance queries.
- The goal is only presence/absence indexing without abundance information — fimpera's
  benchmarking here targets counting structures for k-mer abundances.
- Index scale exceeds available memory even after fimpera's 100x reduction and petabyte-scaled
  k-mer indexing with abundance is required, as no existing method scales to that.
tools:
- name: fimpera
  role: Wraps a counting Bloom filter to amplify s-mer queries (index with fimpera_index,
    query with fimpera_query) and benchmark false-positive rate, space, and query
    time against the raw cBF
  repo: https://github.com/lrobidou/fimpera
- name: Python
  role: Computes sum-of-squared-error overestimation scores and plots the size-vs-score
    trade-off figure
- name: KMC
  role: Provides the k-mer abundance file that fimpera_index consumes for indexing
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:aeb6f33e4c40561398a434105739662e3c0ac446c19b4891a035eea5b53bd84d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - Python
  - fimpera
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# approximate-membership-query-benchmarking

## Summary

Benchmark fimpera-wrapped counting Bloom filters against raw cBFs by measuring false-positive/overestimation scores (sum-of-squared overestimation errors) and space trade-offs, and query time as z increases. Demonstrates that fimpera achieves equal precision at two orders of magnitude less filter size with no query-time penalty.

## When to use

Use when comparing an Approximate Membership Query data structure (e.g., a counting Bloom filter) against its fimpera-wrapped version on abundance queries, to quantify false-positive rate, overestimation score, space, and query time.

## When NOT to use

- The data structure is a plain AMQ without counts (no abundance support), since fimpera requires abundance queries.
- The goal is only presence/absence indexing without abundance information — fimpera's benchmarking here targets counting structures for k-mer abundances.
- Index scale exceeds available memory even after fimpera's 100x reduction and petabyte-scaled k-mer indexing with abundance is required, as no existing method scales to that.

## Inputs

- KMC-format text file of k-mers with abundances (one k-mer, tab, abundance per line)
- Query reads in FASTQ or FASTA (gzipped or not), e.g., first 3,000,000 reads of TARA Oceans sample ERR4691696
- Pre-built counting Bloom filter index of TARA k-mers
- Ground-truth original abundances for the indexed k-mers

## Outputs

- Per-read average abundance outputs from fimpera-wrapped and raw cBF queries
- Tabulated overestimation (sum-of-squared-errors) score versus filter size in bits for both methods
- Query wall-clock time versus z parameter table (Table 2 analogue)
- Space-vs-score trade-off plot (e.g., fig5_sse_tradeoff.png)

## How to apply

Index k-mer abundances from a KMC-format file (k-mer, tab, abundance per line) using fimpera_index with a counting Bloom filter, choosing k, the number of bits per bucket (-b, e.g., 8 for the space trade-off analysis or 5 in the toy example), and the amplification parameter z (s-mers of size k-z stored in the filter). Query a read set (e.g., the first 3,000,000 reads of TARA Oceans sample ERR4691696 in FASTQ) both through the raw cBF and through fimpera at increasing z values, storing per-read average abundances with original (non-log) abundance storage. Compute the sum-of-squared-error overestimation score by accumulating only squared differences where the reported abundance exceeds the original ground truth, per filter size, and tabulate size in bits versus score. Repeat the query timing across z values against the raw-cBF baseline to verify fimpera does not add query-time overhead. Plot size-vs-score trade-off and time-vs-z tables (e.g., saving the space comparison as fig5_sse_tradeoff.png).

## Related tools

- **fimpera** (Wraps a counting Bloom filter to amplify s-mer queries (index with fimpera_index, query with fimpera_query) and benchmark false-positive rate, space, and query time against the raw cBF) — https://github.com/lrobidou/fimpera
- **Python** (Computes sum-of-squared-error overestimation scores and plots the size-vs-score trade-off figure)
- **KMC** (Provides the k-mer abundance file that fimpera_index consumes for indexing)

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3 && ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- At overestimation score 200000, fimpera requires ~3.10×10^9 bits (~3.87×10^8 slots of 8 bits) while the raw cBF requires ~330.9×10^9 bits — a ~100× space gap must be reproduced.
- Recomputed timings must show query time equal to or slightly decreasing as z increases (k-mers spanning an absent s-mer can be skipped).
- The false positive rate of the fimpera-wrapped cBF must be roughly two orders of magnitude lower than the raw cBF at matched precision.
- Reported abundances from fimpera must match or improve the precision of raw cBF abundances against ground truth.
- fimpera must introduce no memory overhead relative to the raw cBF.

## Limitations

- The fimpera implementation presented uses a counting Bloom filter; adaptation to other AMQs requires implementing template specializations (surjective abundance-to-identifier functions).
- fimpera needs a file containing the abundance of each k-mer to index them; KMC can provide such a file, but another k-mer counter must be used if not.
- K-mers whose complexity is too low are reported with abundance -1 in query output.
- No existing method scales up to petabyte-scaled datasets for indexing k-mers with abundance.

## Evidence

- [abstract] False positive rate reduction: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [abstract] No memory overhead / query time: "fimpera does not introduce any memory overhead and may even reduces the query time."
- [readme] fimpera scope and AMQ requirement: "fimpera is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
- [readme] Input formats and indexing: "It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not)."
- [readme] Toy index command: [evidence span withheld — anchor status: external_doc]
