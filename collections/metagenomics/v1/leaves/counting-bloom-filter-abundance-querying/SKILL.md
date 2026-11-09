---
name: counting-bloom-filter-abundance-querying
description: Use when you need per-k-mer abundance queries from large read sets (e.g.
  metagenomic fastq) against a counting Bloom filter index, and want reduced false
  positives via the s-mer amplification parameter z.
when_to_use_negative:
- Your AMQ does not support abundance queries — fimpera is defined for Approximate
  Membership Query structures with counts
- You need exact abundances rather than cBF-estimated abundances with improved precision;
  use an exact k-mer hash table or the KMC counts directly
- The data are not biological sequence reads/k-mers (fastq/fasta or k-mer count files)
  — the provided fimpera implementation targets k-mer indexing of such files
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Indexes k-mer/abundance files into a counting Bloom filter (fimpera_index)
    and answers per-position abundance queries on reads (fimpera_query) using amplified
    s-mer queries parameterized by z
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Produces the k-mer-with-abundance text file that fimpera_index consumes (alternative
    k-mer counters are allowed)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:ef092df5616dbac6e2037a8c88de91df5b3a195fa2a0de702570470b1d6b96e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - fimpera
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# counting-bloom-filter-abundance-querying

## Summary

Index k-mers with abundances into a counting Bloom filter and query biological sequences (fastq/fasta) with fimpera, which wraps the cBF with amplified s-mer queries to cut false positives without memory overhead or slower queries.

## When to use

Use when you need per-k-mer abundance queries from large read sets (e.g. metagenomic fastq) against a counting Bloom filter index, and want reduced false positives via the s-mer amplification parameter z.

## When NOT to use

- Your AMQ does not support abundance queries — fimpera is defined for Approximate Membership Query structures with counts
- You need exact abundances rather than cBF-estimated abundances with improved precision; use an exact k-mer hash table or the KMC counts directly
- The data are not biological sequence reads/k-mers (fastq/fasta or k-mer count files) — the provided fimpera implementation targets k-mer indexing of such files

## Inputs

- KMC-style text file of indexed k-mers with abundances (each line: k-mer, tab, abundance)
- Query read set in fastq or fasta format (gzipped or not), e.g. the first 3,000,000 reads of TARA Oceans sample ERR4691696

## Outputs

- fimpera index file (e.g. index.idx) backed by a counting Bloom filter
- Per-read, per-position k-mer abundance output for the query file (0, positive abundances, or -1 for low-complexity k-mers)
- Query wall-clock timings tabulated against z alongside a raw-cBF baseline (Table 2 style)

## How to apply

Build a k-mer/abundance table with KMC (one line per k-mer: k-mer, tab, abundance), then run fimpera_index with parameters for the number of bits per bucket (-b, e.g. 5), k (e.g. 35), z (number of extra s-mers per k-mer, e.g. 3, so s = k - z), and optionally --canonical, producing an index file (e.g. index.idx). Query reads with fimpera_query, which prints each read with per-position k-mer abundances (-1 marks k-mers of too low complexity). Increasing z amplifies the queried s-mer sets: the paper shows this reduces false positives by two orders of magnitude, and query time does not increase (it slightly decreases as z rises, per Table 2), because k-mers spanning an absent s-mer can be skipped. The index is a counting Bloom filter, so fimpera introduces no memory overhead relative to the raw cBF.

## Related tools

- **fimpera** (Indexes k-mer/abundance files into a counting Bloom filter (fimpera_index) and answers per-position abundance queries on reads (fimpera_query) using amplified s-mer queries parameterized by z) — https://github.com/lrobidou/fimpera
- **KMC** (Produces the k-mer-with-abundance text file that fimpera_index consumes (alternative k-mer counters are allowed))

## Examples

```
./bin/fimpera_index tests/unit/data/1000LinesTest.txt index.idx 1000000 -k 35 -b 5 -z 3; ./bin/fimpera_query index.idx tests/unit/data/exemple_query.fasta
```

## Evaluation signals

- Query time is equal to or slightly decreasing as z increases, matching the raw counting Bloom filter baseline (Table 2 behavior) with no added memory overhead
- False positive abundance calls drop by roughly two orders of magnitude relative to the raw cBF at equal index size
- Index file (e.g. index.idx) is produced and fimpera_query returns one abundance vector per read with one value per k-mer position
- Low-complexity k-mers are flagged as -1 in the output, and non-zero abundances are plausible (toy example shows a single k-mer 'AAAGAGCTGGAACGTGAAAATCGTGAACTGCGCCG' with abundance 13)
- Precision of reported abundances improves relative to querying the raw cBF directly

## Limitations

- k-mers whose complexity is too low are reported as -1 (unusable abundance), as in the README toy example output
- Reported abundances are estimates from the counting Bloom filter, not exact counts, even though fimpera improves their precision
- The fimpera implementation provided uses a counting Bloom filter; adapting other AMQs requires working with the templated fimpera-lib headers
- z trades amplification against s-mer size (s = k - z); the README notes the false-positive/no-drawback claims were flagged as TODO to verify, and the paper's reproduce-results section is a TODO in the README

## Evidence

- [abstract] fimpera does not introduce any memory overhead and may even reduces the query time.: "fimpera does not introduce any memory overhead and may even reduces the query time."
- [abstract] Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances.: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude"
- [intro] It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not).: "It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not)."
- [readme] fimpera needs a file containing the abundance of each k-mers to index them (each line should contain a k-mer (not a s-mer), one tab and the abundance associated with that k-mer). KMC can provide such file: [evidence span withheld — anchor status: not_found]
- [readme] Index a simple KMC output file, using 1000000 bits, kmer of size 35, 5 encoding bits in the counting bloom filter and z=3 (so storing smers of size 32 in the bloom filter).: [evidence span withheld — anchor status: external_doc]
- [readme] The abundance is displayed as -1 if the complexity of the kmer is too low.: [evidence span withheld — anchor status: external_doc]
