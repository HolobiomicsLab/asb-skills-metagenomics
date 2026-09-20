---
name: kmer-to-smer-decomposition
description: Use when implementing or testing approximate membership queries with
  abundances over k-mers and you need to reduce false positive calls by indexing s-mers
  instead of full k-mers.
when_to_use_negative:
- The task only needs presence/absence of k-mers without abundances and petabyte-scale
  indexing is not the bottleneck
- The input is not k-mer counts from a k-mer counter (fimpera requires a file with
  one k-mer, one tab, and its abundance per line — not raw reads — for indexing)
- 'You need an exact k-mer abundance: fimpera reports estimated abundances that can
  be overestimated due to construction false positives'
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: fimpera
  role: Indexes k-mer abundances as s-mer abundances in a counting Bloom filter and
    queries k-mers via z+1 windowed minimum s-abundances
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Produces the k-mer count file (k-mer, tab, abundance per line) consumed by
    fimpera_index
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:5abc2c20c8dfe45a1df2e4aa1c7e6d910bc87ce02531b56754b309c8a75bae52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - fimpera
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kmer-to-smer-decomposition

## Summary

Reconstructs fimpera's s-mer indexing/query strategy: every k-mer (k≥s>0) is split into its k−s+1 constituent s-mers whose abundances are stored in a counting AMQ (counting Bloom filter), and a k-mer query is answered only when all of its s-mers are found, so k-mer-level false positives require multiple s-mer-level false positives.

## When to use

Use when implementing or testing approximate membership queries with abundances over k-mers and you need to reduce false positive calls by indexing s-mers instead of full k-mers.

## When NOT to use

- The task only needs presence/absence of k-mers without abundances and petabyte-scale indexing is not the bottleneck
- The input is not k-mer counts from a k-mer counter (fimpera requires a file with one k-mer, one tab, and its abundance per line — not raw reads — for indexing)
- You need an exact k-mer abundance: fimpera reports estimated abundances that can be overestimated due to construction false positives

## Inputs

- KMC k-mer count file (k-mer + tab + abundance per line), e.g. tests/unit/data/1000LinesTest.txt
- Nucleotide query file (fasta or fastq, gzipped or not), e.g. tests/unit/data/exemple_query.fasta

## Outputs

- fimpera index file (counting Bloom filter over s-mers, e.g. index.idx)
- Per-read k-mer abundance calls for each query read (abundance shown under each base; -1 when k-mer complexity is too low)

## How to apply

Generate small toy nucleotide sequences (fasta/fastq) and produce a k-mer count file with KMC (each line: k-mer, tab, abundance). Build the index by splitting every k-mer into its k−s+1 s-mers (e.g. k=35, z=3, hence s=32) and inserting into a counting Bloom filter the maximum abundance among the k-mers containing each s-mer. On query, for each k-mer average over z+1 consecutive windows and return the minimum s-abundance as the estimated k-mer abundance; report a k-mer as found only if all its s-mers are found. The rationale is that a false positive (resp. overestimation) on a k-mer then requires k−s+1>1 false positives (resp. overestimations) on its s-mers, which reduces the counting Bloom filter false positive rate by two orders of magnitude without memory overhead. Validate with unit tests comparing reported abundances against ground-truth KMC counts on the toy data.

## Related tools

- **fimpera** (Indexes k-mer abundances as s-mer abundances in a counting Bloom filter and queries k-mers via z+1 windowed minimum s-abundances) — https://github.com/lrobidou/fimpera
- **KMC** (Produces the k-mer count file (k-mer, tab, abundance per line) consumed by fimpera_index)

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3 && ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Every unit test in the project's CTest/CI harness passes (repo CI badge runs ci.yml with codecov)
- Reported k-mer abundances on toy data match ground-truth KMC counts (e.g. k-mer AAAGAGCTGGAACGTGAAAATCGTGAACTGCGCCG reported with abundance 13)
- A false positive on a queried k-mer requires k−s+1>1 false positives at the s-mer level — check the FP-rate reduction is roughly two orders of magnitude
- Indexed s-abundance for each s-mer equals the maximum abundance among k-mers containing it
- Low-complexity k-mers are returned as -1 rather than a spurious abundance

## Limitations

- fimpera needs a precomputed k-mer abundance file (one k-mer + tab + abundance per line); it does not count k-mers itself (KMC or another counter is required)
- Construction false positives/overestimations at the s-mer level propagate to k-mer-level overestimations of abundance
- Low-complexity k-mers are returned with abundance -1 rather than an estimate
- The proposed algorithm does not introduce memory overhead but query speedups and FP reductions were shown for counting Bloom filters; adaptation to other AMQs requires templates

## Evidence

- [abstract] fimpera reduces the false positive rate of counting Bloom filters by two orders of magnitude and improves precision of reported abundances: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [full_text] s-mer indexing and query mechanism with FP amplification: [evidence span withheld — anchor status: not_found]
- [methods] query averaging over windows: [evidence span withheld — anchor status: not_found]
- [intro] counting Bloom filter implementation: "The `fimpera` implementation proposed here uses a counting Bloom filter."
- [readme] indexing command and parameters: [evidence span withheld — anchor status: external_doc]
- [readme] input file format requirement: [evidence span withheld — anchor status: external_doc]
- [readme] low complexity marker: [evidence span withheld — anchor status: external_doc]
