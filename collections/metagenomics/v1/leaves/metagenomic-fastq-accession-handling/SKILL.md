---
name: metagenomic-fastq-accession-handling
description: Use when reproducing a k-mer abundance indexing/AMQ benchmark that requires
  concrete public metagenomic FASTQ runs — one dataset to build the index and an independent
  query dataset for FPR measurement.
when_to_use_negative:
- The analysis targets a single sample only — without a separate query FASTQ there
  is no independent FPR estimate to compute.
- The index/query data are non-genomic or non-sequence files (e.g. plain tabular data);
  fimpera indexes k-mers from biological sequences (fastq or fasta, gzipped or not).
- The task requires exact k-mer abundance lookup with no false positives and memory
  constraints make a cAMQ inappropriate (no existing method scales to petabyte-scaled
  datasets with abundance, but exact structures are a different tradeoff).
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: fimpera
  role: Builds the counting Bloom filter index from the ERR1726642 k-mer abundance
    file and queries it with the ERR4691696 FASTQ at thresholds z=0..20
  repo: https://github.com/lrobidou/fimpera
- name: KMC
  role: Produces the k-mer/tab/abundance file that fimpera_index consumes, computed
    from the indexing FASTQ
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad305
    title: '<tt>fimpera</tt>: drastic improvement of Approximate Membership Query
      data-structures with counts'
  source_content_hash: sha256:c3c9de135d341f5b98fbefbb74e489b280e442fc7c16579aedc8c7fe698051ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lrobidou/fimpera
  tools:
  - KMC
  - fimpera
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-fastq-accession-handling

## Summary

Retrieves and stages specific metagenomic FASTQ accessions (e.g. TARA Oceans ERR1726642 for indexing and ERR4691696 for querying) so that counting-Bloom-filter indices and downstream fimpera false-positive-rate evaluations run on reproducible, published data.

## When to use

Use when reproducing a k-mer abundance indexing/AMQ benchmark that requires concrete public metagenomic FASTQ runs — one dataset to build the index and an independent query dataset for FPR measurement.

## When NOT to use

- The analysis targets a single sample only — without a separate query FASTQ there is no independent FPR estimate to compute.
- The index/query data are non-genomic or non-sequence files (e.g. plain tabular data); fimpera indexes k-mers from biological sequences (fastq or fasta, gzipped or not).
- The task requires exact k-mer abundance lookup with no false positives and memory constraints make a cAMQ inappropriate (no existing method scales to petabyte-scaled datasets with abundance, but exact structures are a different tradeoff).

## Inputs

- TARA metagenomics FASTQ run accession ERR1726642 (indexing/reference sample, gzipped fastq accepted)
- TARA metagenomics FASTQ run accession ERR4691696 (query sample)
- KMC-style k-mer abundance text file (one k-mer, tab, abundance per line) derived from the indexing FASTQ

## Outputs

- fimpera counting Bloom filter index file (e.g. index.idx) built from ERR1726642 with k=31, b=5, z=3
- Per-read/per-k-mer abundance query results from ERR4691696 at z = 0..20
- fpr_table.csv with 21 rows (z=0..20) of false positive rates compared against ground-truth k-mer abundances

## How to apply

Fetch the two TARA metagenomic FASTQ accessions named in the reproduce-commands deposit: ERR1726642 to index (source of k-mer abundances) and ERR4691696 as the query set. Index ERR1726642 by first counting its k-mers (e.g. with KMC, since fimpera expects a file of k-mer/tab/abundance lines) and building a counting Bloom filter with fimpera using k=31, b=5 bits per bucket, log2-transformed abundances, and default z=3. Query the resulting index with the ERR4691696 FASTQ at each abundance threshold z from 0 to 20. The two-dataset split (index vs. query) is what makes the false-positive-rate measurement meaningful: called k-mers/reads are compared against ground-truth k-mer abundances to estimate FPR, including the expected ~0.4%/0.56% at z=3 versus 25% for the raw cBF and near-100% FPR at z=20.

## Related tools

- **fimpera** (Builds the counting Bloom filter index from the ERR1726642 k-mer abundance file and queries it with the ERR4691696 FASTQ at thresholds z=0..20) — https://github.com/lrobidou/fimpera
- **KMC** (Produces the k-mer/tab/abundance file that fimpera_index consumes, computed from the indexing FASTQ)

## Examples

```
./bin/fimpera_index kmers_ERR1726642.txt index.idx -k 31 -b 5 -z 3 && ./bin/fimpera_query index.idx ERR4691696.fastq.gz
```

## Evaluation signals

- fpr_table.csv exists and contains exactly 21 rows covering z = 0..20.
- FPR at the raw-cBF setting (z=0) is ~25%, while fimpera at z=3 yields ~0.4%–0.56% FPR using the same memory footprint.
- FPR approaches ~100% at z=20, matching Table 2 of the paper.
- Index construction used the documented parameters k=31, b=5 bits per bucket, log2-transformed abundances, z=3.
- Query calls reproduce paper findings: fimpera reduces the cBF false positive rate by two orders of magnitude.

## Limitations

- k-mers with low sequence complexity are reported with abundance -1 rather than a measured value.
- fimpera requires an external k-mer abundance file (each line: k-mer, tab, abundance); FASTQ files cannot be indexed directly without this counting step.
- Reported FPR values (e.g. ~0.4% vs ~0.56%) depend on the specific index/query sample pair and parameter choices, so exact numbers vary with the data.
- The README marks the 'Reproduce paper results' section as TODO, so the exact reproduce-commands deposit must be located separately.

## Evidence

- [abstract] Findings - fimpera reduces the false positive rate of counting Bloom filters by two orders of magnitude: "Applied to counting Bloom filters, our proposed algorithm reduces the false positive rate by two orders of magnitude and it improves the precision of the reported abundances."
- [readme] README indexing command with k, b, z parameters: [evidence span withheld — anchor status: external_doc]
- [readme] README required k-mer abundance file format: [evidence span withheld — anchor status: external_doc]
- [readme] README accepted input sequence formats: "It proposes a way to index and query k-mers from biological sequences (fastq or fasta, gzipped or not)."
- [readme] README low-complexity k-mer abundance convention: [evidence span withheld — anchor status: external_doc]
