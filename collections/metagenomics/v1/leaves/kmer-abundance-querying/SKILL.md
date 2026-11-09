---
name: kmer-abundance-querying
description: Use when abundances of indexed k-mers must be queried from large (e.g.
  metagenomic/TARA-scale) sequence datasets with low false positive rates and no memory
  overhead.
when_to_use_negative:
- Only presence/absence (not abundance) of k-mers is needed — fimpera targets counting
  AMQs, and a plain AMQ may suffice
- The available data structure does not support abundance queries, since fimpera requires
  a counting AMQ (e.g. a counting Bloom filter) as its backend
- A plain, uncounted k-mer set from KMC is provided directly without converting to
  the tab-separated k-mer/abundance format fimpera requires
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: fimpera
  role: Indexes k-mer abundances as s-mer abundances in a counting Bloom filter (fimpera_index)
    and queries sequences to report per-k-mer abundances with reduced false positives
    (fimpera_query)
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Produces the k-mer count file (k-mer + tab + abundance per line) that fimpera_index
    consumes for indexing
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:ec3a741f54715293602ea5647b66f3a551dcc25412b32603365428510432ffdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - fimpera
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kmer-abundance-querying

## Summary

Index and query k-mer abundances from biological sequences via the fimpera strategy, which splits k-mers into s-mers stored in a counting Bloom filter (cAMQ) and queries by averaging over z+1 consecutive windows to drastically reduce false positives.

## When to use

Use when abundances of indexed k-mers must be queried from large (e.g. metagenomic/TARA-scale) sequence datasets with low false positive rates and no memory overhead.

## When NOT to use

- Only presence/absence (not abundance) of k-mers is needed — fimpera targets counting AMQs, and a plain AMQ may suffice
- The available data structure does not support abundance queries, since fimpera requires a counting AMQ (e.g. a counting Bloom filter) as its backend
- A plain, uncounted k-mer set from KMC is provided directly without converting to the tab-separated k-mer/abundance format fimpera requires

## Inputs

- KMC k-mer count file (.txt, one k-mer per line followed by a tab and its abundance)
- Query biological sequences in fasta or fastq format (gzipped or not)
- Index parameters: k (k-mer size, e.g. 31 or 35), b (bits per cBF bucket, e.g. 5), z (window averaging threshold, default 3), optional --canonical and abundance-to-identifier strategy (e.g. log2 grouping)

## Outputs

- fimpera index file (e.g. index.idx) built on the counting Bloom filter
- Per-read per-k-mer abundance query output: an abundance value under each base, with -1 marking low-complexity k-mers
- Downstream tables such as false-positive-rate curves (fpr_table.csv, fig4_curve.csv) comparing fimpera vs raw cBF

## How to apply

Build a k-mer count file with KMC (one tab-separated k-mer and abundance per line), then run fimpera_index with parameters such as k=31 or 35, b=5 bits per bucket, and z=3, so each k-mer's abundance is associated with its k−s+1 constituent s-mers in a counting Bloom filter (optionally with log2-transformed abundance grouping). Query sequence files (fastq/fasta, gzipped or not) with fimpera_query, which reports an estimated abundance per k-mer by averaging over z+1 consecutive windows and taking the minimum s-abundance; a k-mer is reported only if all its s-mers are found, so a false positive requires k−s+1>1 s-mer-level false positives. A reported abundance of -1 flags k-mers whose complexity is too low. Because fimpera wraps any cAMQ supporting abundance queries via templates, alternative counting structures can be substituted without changing the workflow.

## Related tools

- **fimpera** (Indexes k-mer abundances as s-mer abundances in a counting Bloom filter (fimpera_index) and queries sequences to report per-k-mer abundances with reduced false positives (fimpera_query)) — https://github.com/lrobidou/fimpera
- **KMC** (Produces the k-mer count file (k-mer + tab + abundance per line) that fimpera_index consumes for indexing)

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3 && ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Query output on toy data (e.g. tests/unit/data/exemple_query.fasta) reports exactly the indexed abundance, e.g. k-mer AAAGAGCTGGAACGTGAAAATCGTGAACTGCGCCG with abundance 13
- Wrapping a cBF designed for 25% false positive calls with fimpera (z=3) yields an FPR of about 0.4–0.56% instead of 25% at equal space
- At z=3, achieving 2% FPR takes only ~2.98×10^9 bits with fimpera versus ~65.5×10^9 bits for the raw cBF (~21× less space)
- Low-complexity k-mers are consistently reported with abundance -1 in the query output
- Unit tests comparing reported abundances against ground-truth KMC counts on toy sequences all pass

## Limitations

- fimpera requires a precomputed k-mer abundance file (e.g. from KMC); raw fastq cannot be indexed directly
- Reported abundance is -1 for k-mers whose complexity is too low, so low-complexity regions yield no abundance estimate
- The provided implementation uses a counting Bloom filter; adaptation to other AMQs requires template/header work described in lib/include/fimpera-lib/README.md

## Evidence

- [other] fimpera indexes by splitting every k-mer into its constituent s-mers: [evidence span withheld — anchor status: not_found]
- [readme] README index invocation: [evidence span withheld — anchor status: external_doc]
- [other] false positive reduction requires multiple s-mer errors: "a false positive (resp. overestimation) on a k-mer requires k−s+1>1 false positives (resp. overestimations) on its s-mers"
- [readme] README abundance file requirement: [evidence span withheld — anchor status: external_doc]
