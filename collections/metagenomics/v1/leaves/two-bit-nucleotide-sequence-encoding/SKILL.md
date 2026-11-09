---
name: two-bit-nucleotide-sequence-encoding
description: Use when implementing k-mer hashing or maximal exact match (MEM) detection
  over FASTA reads/genomes where memory-efficient nucleotide representation and bit-space
  coordinate thresholds are needed, as in the pal-MEM repeat-detection step of Palidis.
when_to_use_negative:
- Input reads lack ITR-containing insertion sequences — Palidis is 'only equipped
  with discovering ISs with ITRs', so the encoding pipeline cannot find ITR-lacking
  ISs
- 'The task needs ITRs shorter than the k-mer-supported thresholds: the ISC catalogue
  contains ITRs of 25 or more nucleotides although ITRs can be as short as ten nucleotides'
- Input is protein or quality-trimmed variable-length sequences where a 2-bit per
  nucleotide alphabet (ACGT only) is insufficient
tools:
- name: pal-MEM
  role: Performs the two-bit-encoded k-mer hashing and maximal exact matching against
    query reads, implementing the encoding and boundary conditions
  repo: github.com/blue-moon22/pal-MEM
- name: Palidis
  role: Pipeline whose Step 1/2 (convertToFasta, palmem) invokes the encoded MEM detection
    to recover repeat sequences for ITR discovery
  repo: github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:c5b819d9da7b818b5332919ee0d60f8625db32cb7b03ea17975dc02e3ddaacca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Palidis
  - pal-MEM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# two-bit-nucleotide-sequence-encoding

## Summary

Compact two-bit encoding (2 bits per nucleotide) of reference k-mers as used by Palidis/pal-MEM, enabling hash-table MEM lookup over metagenomic reads and letting coordinate filters operate in bit space (e.g. a 20-nucleotide edge buffer = 40 bits).

## When to use

Use when implementing k-mer hashing or maximal exact match (MEM) detection over FASTA reads/genomes where memory-efficient nucleotide representation and bit-space coordinate thresholds are needed, as in the pal-MEM repeat-detection step of Palidis.

## When NOT to use

- Input reads lack ITR-containing insertion sequences — Palidis is 'only equipped with discovering ISs with ITRs', so the encoding pipeline cannot find ITR-lacking ISs
- The task needs ITRs shorter than the k-mer-supported thresholds: the ISC catalogue contains ITRs of 25 or more nucleotides although ITRs can be as short as ten nucleotides
- Input is protein or quality-trimmed variable-length sequences where a 2-bit per nucleotide alphabet (ACGT only) is insufficient

## Inputs

- Reference genome/contig sequences in FASTA (k-mer source for the hash table)
- Query metagenomic reads in FASTA, including paired-end FASTA files (-f1/-f2) or a single FASTA (-fu)

## Outputs

- Hash-table-backed MEM records: tab-delimited file (e.g. mems_end_buffered.tsv) with read identifier, MEM start, MEM end, and length
- pal-MEM outputs: _IR.tab of read pairs with ITR coordinates (LCoord/RCoord), _IR fasta files of reads with inverted repeats, and _discord_non_IR fasta files

## How to apply

Load the reference sequences and encode each k-mer using two-bit (2 bits per nucleotide) encoding, then populate a hash table mapping each encoded k-mer to its reference positions; position j is stored only when the boundary condition br <= j((L-k)+1) <= er-k+1 holds. Scan the query metagenomic reads and perform maximal exact matching against this k-mer hash table using a pal-MEM-style MEM algorithm (derived from E-MEM), with defaults such as k-mer length 15 and minimum match length 24 adjustable via options (-k, -l). Because coordinates are in bit space, thresholds expressed in nucleotides must be doubled: the end-buffer filter discards any MEM whose start or end lies within 20 nucleotides, i.e. 40 bits in the two-bit encoding, of either read edge to suppress technical repeats from amplified libraries. Surviving MEM records (read identifier, MEM start, MEM end, length) are written out for downstream candidate-ITR filtering.

## Related tools

- **pal-MEM** (Performs the two-bit-encoded k-mer hashing and maximal exact matching against query reads, implementing the encoding and boundary conditions) — github.com/blue-moon22/pal-MEM
- **Palidis** (Pipeline whose Step 1/2 (convertToFasta, palmem) invokes the encoded MEM detection to recover repeat sequences for ITR discovery) — github.com/blue-moon22/palidis4

## Examples

```
pal-mem -f1 example_1.fasta -f2 example_1.fasta -o example -l 30 -m 50 -k 18 -t 8
```

## Evaluation signals

- Verify mems_end_buffered.tsv exists and is a valid TSV with read identifier and MEM-coordinate fields present
- Confirm no retained MEM start or end position falls inside the 20-nucleotide (40-bit) end buffer at either read edge
- Check every stored hash-table position satisfies the boundary condition br <= j((L-k)+1) <= er-k+1
- pal-MEM _IR.tab output should contain read names with LCoord and RCoord fields, and length-filtered matches should respect -l/-m bounds (e.g. 30-50 in the README example)
- Reported bit-space coordinates must equal 2x the nucleotide-space offsets for the same match

## Limitations

- Palidis is currently only equipped with discovering ISs with ITRs; ISs lacking ITRs are missed
- Running with lower minimum ITR length thresholds and smaller k-mer length is more computationally intensive, especially with complex mixed microbial genomes
- pal-MEM requires a 64-bit system plus Boost and cmake, and its supplied binaries were built on Mac OS (Catalina v10.15.6) and Linux
- ISs flanked by ITRs beyond the 500 to 3000 bp proximity range (e.g. lengthened by nested insertion events) can be missed downstream of MEM detection

## Evidence

- [methods] Load the reference sequences and encode each k-mer using two-bit (2 bits per nucleotide) encoding.: [evidence span withheld — anchor status: not_found]
- [methods] End-buffer filter expressed in bit space: [evidence span withheld — anchor status: not_found]
- [readme] pal-MEM purpose and E-MEM lineage: [evidence span withheld — anchor status: external_doc]
- [readme] pal-MEM requirements: "- pal-MEM requires a 64-bit system
- [Boost](https://www.boost.org/)
- [cmake](https://cmake.org/download/)"
- [intro] Palidis uses efficient maximal exact matching: "we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm"
