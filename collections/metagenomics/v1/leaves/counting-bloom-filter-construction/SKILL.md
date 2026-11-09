---
name: counting-bloom-filter-construction
description: Use when building a counting Approximate Membership Query (cAMQ) index
  from k-mer abundance files (e.g. KMC output) where k-mers must be indexed as s-mers
  with counts, typically with k=31 or 35, b=5, and z=3.
when_to_use_negative:
- Input is raw FASTA/FASTQ reads rather than a k-mer abundance file — fimpera_index
  requires a pre-counted k-mer file (e.g. from KMC) as input.
- The task is presence/absence membership without counts, where a plain (non-counting)
  Bloom filter or exact k-mer set would suffice and fimpera's s-mer machinery adds
  complexity.
- The downstream query tool cannot consume fimpera's index format or you need a different
  AMQ without adapting the templated fimpera-lib interfaces.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: fimpera
  role: Builds the counting Bloom filter index by splitting each k-mer into its s-mers
    and inserting s-abundances (fimpera_index); the implementation's AMQ is a counting
    Bloom filter configured via -b bits per bucket.
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Produces the k-mer abundance text file (k-mer, tab, abundance per line) that
    fimpera_index consumes; any other k-mer counter program may substitute.
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:e513d16336fde80f5cf048c1dea0b6a43c0a3b98134865e13244415709dc2462
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - counting Bloom filter
  - fimpera
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# counting-bloom-filter-construction

## Summary

Constructs a counting Bloom filter (cBF) index of s-mer abundances derived from k-mer counts, using fimpera's indexing step (k≥s>0) so that the index can later be wrapped by fimpera for low-false-positive abundance queries.

## When to use

Use when building a counting Approximate Membership Query (cAMQ) index from k-mer abundance files (e.g. KMC output) where k-mers must be indexed as s-mers with counts, typically with k=31 or 35, b=5, and z=3.

## When NOT to use

- Input is raw FASTA/FASTQ reads rather than a k-mer abundance file — fimpera_index requires a pre-counted k-mer file (e.g. from KMC) as input.
- The task is presence/absence membership without counts, where a plain (non-counting) Bloom filter or exact k-mer set would suffice and fimpera's s-mer machinery adds complexity.
- The downstream query tool cannot consume fimpera's index format or you need a different AMQ without adapting the templated fimpera-lib interfaces.

## Inputs

- KMC-style k-mer count text file (one k-mer, one tab, one abundance per line)
- Parameters: number of bits in the filter, k (k-mer size, e.g. 31 or 35), b (bits per bucket, e.g. 5), z (window parameter, e.g. 3, giving s = k − z)
- Optional abundance transformation choice (e.g. log2) from abundanceToIdentifierStrategy

## Outputs

- fimpera index file (e.g. index.idx) containing a counting Bloom filter of s-mer abundances

## How to apply

Start from a text file of k-mer abundances (each line: a k-mer, one tab, its abundance) produced by KMC or another k-mer counter; fimpera cannot index directly from FASTA/FASTQ at this stage. Run fimpera_index with an explicit filter size in bits (e.g. 1000000 bits), k-mer size k (e.g. 31 or 35), bits-per-bucket b in the counting Bloom filter (e.g. 5), and the window parameter z (e.g. 3, meaning s-mers of size k−z are stored). Optionally pass --canonical to index canonical k-mers. Optionally wrap abundances with a surjective function such as log2 transformation via the abundanceToIdentifierStrategy classes to compress counts. The index splits every k-mer into its k−s+1 constituent s-mers and stores the s-abundance (the maximum abundance among containing k-mers) in the counting Bloom filter, producing a named index file (e.g. index.idx) with no memory overhead relative to the raw cBF.

## Related tools

- **fimpera** (Builds the counting Bloom filter index by splitting each k-mer into its s-mers and inserting s-abundances (fimpera_index); the implementation's AMQ is a counting Bloom filter configured via -b bits per bucket.) — https://github.com/lrobidou/fimpera
- **KMC** (Produces the k-mer abundance text file (k-mer, tab, abundance per line) that fimpera_index consumes; any other k-mer counter program may substitute.)

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3
```

## Evaluation signals

- The output index file (e.g. index.idx) exists and has the expected filter size in bits as passed to fimpera_index.
- Querying a k-mer known to be indexed (e.g. the toy k-mer AAAGAGCTGGAACGTGAAAATCGTGAACTGCGCCG with abundance 13) returns its stored abundance, and k-mers absent from the index return 0.
- Low-complexity k-mers are reported with abundance -1 by the query path, confirming the complexity guard behaves as documented.
- The index stores s-mers of size k−z (e.g. s=32 when k=35 and z=3), verifiable by checking that queried s-abundances correspond to maxima over containing k-mers.
- Unit tests over toy FASTA/FASTQ and the KMC count file pass, with reported abundances matching ground-truth KMC counts on the toy data.

## Limitations

- fimpera requires a file of k-mer abundances (not raw reads) for indexing; KMC or an equivalent counter must be run first.
- The provided implementation uses a counting Bloom filter as the AMQ; using another data structure requires adapting fimpera's templated headers (see lib/include/fimpera-lib/README.md).
- K-mers whose complexity is too low are reported as -1 during queries rather than an abundance.
- With z=0 the false-positive reduction vanishes (the raw cBF false positive rate, e.g. 25%, is recovered); the recommended default z=3 must be used to obtain the reduced FPR.

## Evidence

- [intro] The `fimpera` implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not).: "The `fimpera` implementation proposed here uses a counting Bloom filter. It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not)."
- [readme] fimpera needs a file containing the abundance of each k-mers to index them (each line should contain a k-mer (not a s-mer), one tab and the abundance: [evidence span withheld — anchor status: external_doc]
- [readme] Index a simple KMC output file, using 1000000 bits, kmer of size 35, 5 encoding bits in the counting bloom filter and z=3: [evidence span withheld — anchor status: external_doc]
- [readme] This file contains definition of multiple surjective functions that links abundances to their values.: [evidence span withheld — anchor status: external_doc]
- [intro] `fimpera` relies on templates, hence it can be easily adapted to any other AMQ that supports abundance queries, for any usage.: "`fimpera` relies on templates, hence it can be easily adapted to any other AMQ that supports abundance queries, for any usage."
