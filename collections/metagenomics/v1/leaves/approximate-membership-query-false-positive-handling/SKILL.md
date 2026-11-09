---
name: approximate-membership-query-false-positive-handling
description: Use when building an approximate membership query index over k-mer abundances
  where false positive k-mer calls or overestimated abundances must be reduced without
  memory overhead.
when_to_use_negative:
- The application only needs presence/absence of k-mers, not abundances — fimpera
  targets cAMQs that support abundance queries
- The dataset is too small to be affected by AMQ false positives, where the added
  z-window averaging machinery is unnecessary
- A non-counting AMQ (plain Bloom filter without abundance support) is the only available
  structure, since fimpera requires abundance queries
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Implements the s-mer indexing and z-window query strategy on top of a counting
    Bloom filter (cAMQ), via fimpera_index and fimpera_query binaries
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Produces the k-mer abundance count file that fimpera_index consumes; any k-mer
    counter emitting 'kmer<TAB>abundance' lines can substitute
- name: sliding-minimum-windows
  role: Benchmark of sliding-window minimum algorithms supporting the in-place fixed-window
    approach used for the z+1 window averaging during queries
  repo: lrobidou/sliding-minimum-windows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:101cc5ae5c1ecda49c95e88cff5536a8ac12bf7242ed07e06ac7835e7ea4ebe7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - fimpera
  - sliding-minimum-windows
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# approximate-membership-query-false-positive-handling

## Summary

fimpera-style s-mer indexing reduces false positives (and abundance overestimations) of counting AMQs such as counting Bloom filters by splitting each indexed k-mer into its k−s+1 constituent s-mers and requiring all s-mer queries to succeed before a k-mer is called present.

## When to use

Use when building an approximate membership query index over k-mer abundances where false positive k-mer calls or overestimated abundances must be reduced without memory overhead.

## When NOT to use

- The application only needs presence/absence of k-mers, not abundances — fimpera targets cAMQs that support abundance queries
- The dataset is too small to be affected by AMQ false positives, where the added z-window averaging machinery is unnecessary
- A non-counting AMQ (plain Bloom filter without abundance support) is the only available structure, since fimpera requires abundance queries

## Inputs

- KMC-style count file with one k-mer per line followed by a tab and its abundance (fimpera_index input, .txt)
- Query file of reads: FASTA or FASTQ, gzipped or not

## Outputs

- fimpera index file (e.g. index.idx) built on s-mers stored in a counting Bloom filter
- Per-read query output listing each base and the estimated abundance of the k-mer starting at that position (−1 when k-mer complexity is too low)

## How to apply

Build a k-mer count file (e.g. with KMC, 'kmer<TAB>abundance' per line), then run fimpera_index choosing k (e.g. 35), a counting-Bloom-filter bits-per-bucket value (e.g. -b 5), and z (e.g. 3), so s-mers of size k−z are stored: each s-mer bucket receives the maximum abundance among containing k-mers (mapped through a surjective abundance function such as log2 grouping). At query time, for each queried k-mer average over z+1 consecutive s-mer windows and return the minimum s-abundance as the estimated k-mer abundance; a k-mer is only reported found if all its s-mers are found, so a k-mer false positive (resp. overestimation) requires k−s+1>1 false positives (resp. overestimations) at the s-mer level. The implementation is templated on the AMQ, so any counting AMQ supporting abundance queries can be substituted for the default counting Bloom filter.

## Related tools

- **fimpera** (Implements the s-mer indexing and z-window query strategy on top of a counting Bloom filter (cAMQ), via fimpera_index and fimpera_query binaries) — https://github.com/lrobidou/fimpera
- **KMC** (Produces the k-mer abundance count file that fimpera_index consumes; any k-mer counter emitting 'kmer<TAB>abundance' lines can substitute)
- **sliding-minimum-windows** (Benchmark of sliding-window minimum algorithms supporting the in-place fixed-window approach used for the z+1 window averaging during queries) — lrobidou/sliding-minimum-windows

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3; ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Queried k-mer abundances match the ground-truth KMC counts on toy data (e.g. the unique indexed k-mer AAAGAGCTGGAACGTGAAAATCGTGAACTGCGCCG reports abundance 13)
- False-positive rate of the counting Bloom filter is reduced by about two orders of magnitude relative to plain cAMQ use
- A k-mer false positive requires k−s+1>1 s-mer false positives — unit tests confirm single spurious s-mer hits do not yield k-mer calls
- Low-complexity k-mers are flagged with abundance −1 rather than being reported as found
- Full unit-test suite (CTest/CI harness at lib/tests, ci.yml workflow) passes on toy sequences and the 1000LinesTest.txt count file

## Limitations

- k-mers with low complexity are returned as −1 and cannot be evaluated
- Applied to counting Bloom filters, fimpera improves FP rate but relies on the underlying cAMQ's bucket counts; bucket collisions still cause s-mer overestimations
- Query output reports -1 abundances for low-complexity k-mers and 0 for absent k-mers, requiring downstream handling of sentinel values

## Evidence

- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [intro] a k-mer is reported as found only if all of its s-mers are found: "`fimpera` is a simple strategy for reducing false positive calls from any Approximate Membership Query data structure (AMQ) that supports abundance queries."
- [readme] fimpera needs a file containing the abundance of each k-mers to index them: [evidence span withheld — anchor status: external_doc]
- [readme] Index a simple KMC output file, using 1000000 bits, kmer of size 35: [evidence span withheld — anchor status: external_doc]
