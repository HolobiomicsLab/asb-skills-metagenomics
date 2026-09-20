---
name: metagenomic-read-coordinate-processing
description: Use when scanning metagenomic FASTA reads with pal-MEM for inverted terminal
  repeats and you need to encode read coordinates in two-bit space, apply edge-boundary
  conditions, or exclude MEMs near read ends that arise from amplification artifacts.
when_to_use_negative:
- Reads are in raw FASTQ.GZ format — Palidis pre-processes these to FASTA first via
  convertToF before coordinate processing
- The target insertion sequences lack inverted terminal repeats (ITRs) — Palidis is
  only equipped to discover ISs with ITRs, so MEM-coordinate filtering will not help
- The goal is de novo assembly or mapping against assemblies (Palidis uses Bowtie2
  steps for that, not coordinate-based MEM filtering)
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: pal-MEM
  role: Performs efficient maximal exact matching on metagenomic reads; its two-bit
    k-mer encoding and MEM coordinate records are the object of the end-buffer filter
  repo: https://github.com/blue-moon22/pal-MEM
- name: Palidis
  role: Pipeline that invokes pal-MEM (palmem step) to obtain repeat sequences from
    read coordinates before distance filtering and ITR clustering
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:bdd5c1434c737e17353931d883d5c9a07b9a03cb0c65d76026c9cab9500971d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/pal-MEM
  tools:
  - Palidis
  - pal-MEM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-read-coordinate-processing

## Summary

Handles read-coordinate manipulation in the pal-MEM step of the Palidis pipeline: k-mers are encoded with two bits per nucleotide, hash-table lookups respect boundary conditions, and MEMs falling inside a 20-nucleotide (40-bit) end buffer are discarded to suppress technical repeats from amplified read libraries.

## When to use

Use when scanning metagenomic FASTA reads with pal-MEM for inverted terminal repeats and you need to encode read coordinates in two-bit space, apply edge-boundary conditions, or exclude MEMs near read ends that arise from amplification artifacts.

## When NOT to use

- Reads are in raw FASTQ.GZ format — Palidis pre-processes these to FASTA first via convertToF before coordinate processing
- The target insertion sequences lack inverted terminal repeats (ITRs) — Palidis is only equipped to discover ISs with ITRs, so MEM-coordinate filtering will not help
- The goal is de novo assembly or mapping against assemblies (Palidis uses Bowtie2 steps for that, not coordinate-based MEM filtering)

## Inputs

- Paired-end or single-end FASTA reads (pre-processed from FASTQ.GZ via Palidis convertToFasta)
- Reference/metagenomic read sequences for two-bit k-mer encoding and MEM scanning

## Outputs

- mems_end_buffered.tsv — tab-delimited records of surviving MEMs with read identifier, MEM start, MEM end, and length
- pal-MEM _IR.tab file with LCoord/RCoord coordinates of inverted terminal repeats per read

## How to apply

Load reference sequences and encode each k-mer using two-bit (2 bits per nucleotide) encoding, then populate a hash table mapping encoded k-mers to reference positions, storing position j only where the boundary condition br <= j((L-k)+1) <= er-k+1 holds. Scan query metagenomic reads and perform maximal exact matching against the k-mer hash table with the pal-MEM-style MEM algorithm (default k-mer length 15 per the pal-MEM README). Apply the end-buffer filter: discard any MEM whose start or end lies within 20 nucleotides (40 bits in the two-bit encoding) of either read edge, which suppresses technical repeats from amplified read libraries. Write surviving MEM records (read identifier, MEM start, MEM end, length) to mems_end_buffered.tsv. The rationale is that amplified libraries generate spurious repeat signal concentrated at read termini, so excluding an edge buffer prevents these artifacts from entering downstream ITR detection.

## Related tools

- **pal-MEM** (Performs efficient maximal exact matching on metagenomic reads; its two-bit k-mer encoding and MEM coordinate records are the object of the end-buffer filter) — https://github.com/blue-moon22/pal-MEM
- **Palidis** (Pipeline that invokes pal-MEM (palmem step) to obtain repeat sequences from read coordinates before distance filtering and ITR clustering) — https://github.com/blue-moon22/palidis4

## Examples

```
pal-mem -f1 example_1.fasta -f2 example_2.fasta -o example -l 30 -m 50 -k 18 -t 8
```

## Evaluation signals

- mems_end_buffered.tsv exists and is a valid TSV with read identifier and MEM coordinate fields present
- No retained MEM has its start or end position falling inside the 20-nucleotide (40-bit) end buffer at either read edge
- Encoded k-mer positions satisfy the boundary condition br <= j((L-k)+1) <= er-k+1 before hash-table insertion
- pal-MEM run completes with the chosen parameters (e.g. -l minimum match length, -k k-mer length) and produces the expected _IR.tab output with LCoord/RCoord pairs

## Limitations

- Technical-repeat suppression depends on the 20-nt edge buffer; repeat signal from other amplification artifacts outside the buffer will be retained
- Running with lower minimum ITR length thresholds and smaller k-mer lengths becomes more computationally intensive, especially with complex mixed microbial genomes
- pal-MEM requires a 64-bit system with Boost and cmake installed; on HPC, library paths must be set explicitly

## Evidence

- [methods] To suppress technical repeats from amplified read libraries, pal-MEM excludes MEMs whose start or end positions lie within a 20-nucleotide (40-bit) buffer of either end of the read.: [evidence span withheld — anchor status: not_found]
- [readme] pal-MEM finds inverted terminal repeats (ITRs) in large genomes, including metagenomes. It is modified from the program E-MEM: [evidence span withheld — anchor status: external_doc]
- [readme] -k set the k-mer length. Default: 15: [evidence span withheld — anchor status: external_doc]
- [intro] we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm: "we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm"
