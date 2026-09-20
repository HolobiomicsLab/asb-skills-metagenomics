---
name: kmer-reference-hash-table-construction
description: Use when implementing or reproducing MEM-based repeat discovery (e.g.
  pal-MEM/E-MEM style) and you need an encoded k-mer-to-reference-position index to
  seed maximal exact matches against metagenomic reads.
when_to_use_negative:
- Input is raw FASTQ reads to be searched, not a reference to index — the hash table
  is built from reference sequences only
- You need alignment-quality mapping of reads to references; the k-mer index seeds
  exact matches (MEMs), not gapped alignments (use Bowtie2 for that in the Palidis
  pipeline)
- Sequences are shorter than the chosen k-mer length, since the boundary condition
  br <= j((L-k)+1) <= er-k+1 leaves no valid positions to store
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: pal-MEM
  role: Implements the two-bit k-mer encoding and hash table used to compute maximal
    exact matches for inverted repeat detection; modified from E-MEM
  repo: https://github.com/blue-moon22/pal-MEM
- name: Palidis
  role: Pipeline that invokes pal-MEM (palmem step) to obtain repeat sequences from
    pre-processed FASTA reads
  repo: https://github.com/blue-moon22/palidis4
- name: E-MEM
  role: Original efficient MEM computation algorithm from which pal-MEM's indexing
    and matching were derived
  repo: https://github.com/lucian-ilie/E-MEM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:98848f790e6af6b9e87cfffcbc621cbe5ae7e2adc4740fcac4f02d49ae07a0e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/pal-MEM
  tools:
  - E-MEM
  - Palidis
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kmer-reference-hash-table-construction

## Summary

Constructs a hash table that maps two-bit-encoded k-mers from a reference genome set to their reference positions, as used inside the pal-MEM maximal exact matching stage of the Palidis insertion sequence discovery pipeline.

## When to use

Use when implementing or reproducing MEM-based repeat discovery (e.g. pal-MEM/E-MEM style) and you need an encoded k-mer-to-reference-position index to seed maximal exact matches against metagenomic reads.

## When NOT to use

- Input is raw FASTQ reads to be searched, not a reference to index — the hash table is built from reference sequences only
- You need alignment-quality mapping of reads to references; the k-mer index seeds exact matches (MEMs), not gapped alignments (use Bowtie2 for that in the Palidis pipeline)
- Sequences are shorter than the chosen k-mer length, since the boundary condition br <= j((L-k)+1) <= er-k+1 leaves no valid positions to store

## Inputs

- Reference genome/metagenome sequences in FASTA format
- k-mer length parameter (pal-MEM default k = 15, settable via -k)

## Outputs

- Hash table mapping two-bit-encoded k-mers to their reference positions
- Encoded reference used downstream by the pal-MEM scan to produce MEM records (e.g. tab-delimited _IR.tab with LCoord/RCoord)

## How to apply

Load the reference sequences and encode each k-mer using two-bit (2 bits per nucleotide) encoding, so a k-mer occupies 2k bits. Populate a hash table mapping each encoded k-mer to its reference position, storing position j only when the boundary condition br <= j((L-k)+1) <= er-k+1 holds, which keeps only k-mers fully inside the valid window of a sequence of length L. In the pal-MEM default configuration k = 15 (settable via the -k option); the hash table is then queried during the scan of query metagenomic reads to seed maximal exact matches used to find inverted terminal repeats. Because MEM seeds derived from reads will later be filtered for technical repeats, keep read-edge handling separate from this reference-index construction step.

## Related tools

- **pal-MEM** (Implements the two-bit k-mer encoding and hash table used to compute maximal exact matches for inverted repeat detection; modified from E-MEM) — https://github.com/blue-moon22/pal-MEM
- **Palidis** (Pipeline that invokes pal-MEM (palmem step) to obtain repeat sequences from pre-processed FASTA reads) — https://github.com/blue-moon22/palidis4
- **E-MEM** (Original efficient MEM computation algorithm from which pal-MEM's indexing and matching were derived) — https://github.com/lucian-ilie/E-MEM

## Examples

```
pal-mem -fu single.fasta -o example -k 15 -l 24 -m 100
```

## Evaluation signals

- Every stored hash entry position j satisfies the boundary condition br <= j((L-k)+1) <= er-k+1 for its sequence of length L
- Encoded k-mer bit length equals 2k (two bits per nucleotide); decoding any stored key round-trips to the expected reference substring
- Downstream MEM scan output is a valid tab-delimited file (e.g. _IR.tab) with sequence names carrying LCoord/RCoord ITR coordinates
- Querying the hash table with k-mers from a reference sequence returns the correct original positions
- No valid in-window k-mer of the reference is missing from the table (spot-check a sample of positions)

## Limitations

- pal-MEM requires a 64-bit system plus Boost and cmake to build, and HPC runs need library paths set explicitly
- The index supports exact matching only; maximal exact matches cannot represent mismatches, so diverged repeats are missed
-  palidis4 discussion notes lower k-mer lengths and shorter ITR thresholds make the whole approach more computationally intensive, especially with complex mixed microbial genomes

## Evidence

- [methods] populate hash table with boundary condition: [evidence span withheld — anchor status: not_found]
- [readme] pal-MEM description: [evidence span withheld — anchor status: external_doc]
- [readme] k-mer option default: [evidence span withheld — anchor status: external_doc]
- [intro] Palidis MEM-based discovery: "we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm"
- [readme] pipeline step ordering: "2. Efficient maximal exact matching to get repeat sequences using [pal-MEM](https://github.com/blue-moon22/pal-MEM) [`palmem`]"
