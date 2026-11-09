---
name: paired-end-reverse-complement-handling
description: Use when classifying paired-end reads (e.g., setB2_1/setB2_2 FASTA files)
  whose strand orientation is unknown, and classification quality depends on considering
  forward and reverse-complement strands.
when_to_use_negative:
- Reads are single-end with no reverse-complement variant required for orientation
  handling
- A classifier already handles strand direction internally and paired-end data structures
  would be redundant
- Only raw reads without a reference genome database to merge against
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Runs ClusterLCP, ClusterBWT_DA, and Classify on both forward and reverse-complement
    merged collections to classify each read pair
  repo: https://github.com/veronicaguerrini/LiME
- name: eGap
  role: Merges ebwt structures of read sets (and their reverse complements) with the
    genome set, computing lcp and da via --lcp and --da options
  repo: https://github.com/felipelouza/egap
- name: BCR
  role: Alternative tool to build ebwt, lcp, and da data structures for read or genome
    collections
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: Alternative external-memory builder of generalized enhanced suffix arrays
    (SA, LCP, BWT) convertible with EGSAtoBCR
  repo: https://github.com/felipelouza/egsa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:a4c84ef8e9ea132252909b57f12d4f942020ecfefcaa5f07ea15bd5889649649
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - LiME
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# paired-end-reverse-complement-handling

## Summary

Handles unknown strand direction in paired-end metagenomic classification (LiME workflow) by building and analyzing data structures for both the original reads and their reverse complements, so each read pair is classified with reads properly oriented.

## When to use

Use when classifying paired-end reads (e.g., setB2_1/setB2_2 FASTA files) whose strand orientation is unknown, and classification quality depends on considering forward and reverse-complement strands.

## When NOT to use

- Reads are single-end with no reverse-complement variant required for orientation handling
- A classifier already handles strand direction internally and paired-end data structures would be redundant
- Only raw reads without a reference genome database to merge against

## Inputs

- Paired-end read FASTA files (e.g., setB2_1.fasta, setB2_2.fasta)
- Reference genome FASTA (Refs.fasta, 930 genomes)
- ebwt, lcp, and da data structures for merged read+genome collections, including reverse-complement variants
- Taxonomic lineage file (semicolon-separated, e.g., ./Datasets/Reference_database.csv)

## Outputs

- Merged data structures for forward and reverse-complement strands (4 collections for a paired-end set)
- ClusterBWT_DA similarity output files for each strand/collection
- Classification records per read pair in C/U/A/H format: C/U/A/H,IdSeqRead,TaxID,maxSim

## How to apply

Since reads' strand direction is unknown, both the original sequences and their reverse complements must be included so reads are kept properly oriented. For each paired-end read collection, build ebwt/lcp/da structures for the read set and its reverse complement (e.g., setB2_1+Refs.fasta, setB2_1_RC+Refs.fasta, setB2_2+Refs.fasta, setB2_2_RC+Refs.fasta), merging read and genome structures with eGap using --lcp and --da. Run Step 1 (ClusterLCP, alpha=16) and Step 2 (ClusterBWT_DA, beta=0.25) separately on all four merged collections, then feed the outputs of Step 2 into Classify together with the lineage file (Reference_database.csv) and TaxRank=1 so each read pair, not each read, receives one C/U/A/H record. The rationale is that paired-end evidence resolves ambiguous entries during re-examination, improving precision over single-strand classification.

## Related tools

- **LiME** (Runs ClusterLCP, ClusterBWT_DA, and Classify on both forward and reverse-complement merged collections to classify each read pair) — https://github.com/veronicaguerrini/LiME
- **eGap** (Merges ebwt structures of read sets (and their reverse complements) with the genome set, computing lcp and da via --lcp and --da options) — https://github.com/felipelouza/egap
- **BCR** (Alternative tool to build ebwt, lcp, and da data structures for read or genome collections) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (Alternative external-memory builder of generalized enhanced suffix arrays (SA, LCP, BWT) convertible with EGSAtoBCR) — https://github.com/felipelouza/egsa

## Examples

```
ClusterLCP setB2_1+Refs.fasta numReads numRefs 16 threads; ClusterBWT_DA setB2_1+Refs.fasta 100 0.25 threads; Classify N Input1 ... InputN numReads numRefs OutputFile ./Datasets/Reference_database.csv 1 threads
```

## Evaluation signals

- All four merged collections exist for a paired-end set: setB2_1+Refs.fasta, setB2_1_RC+Refs.fasta, setB2_2+Refs.fasta, setB2_2_RC+Refs.fasta (including reverse complements)
- Classify emits one C/U/A/H record per read pair; positive-control correct-assignment rate ≈ 0.999 on setB2 (~20M reads)
- Negative control (~5M random shuffled reads) classified rate < 0.0001
- ClusterLCP run with alpha=16 and ClusterBWT_DA with beta=0.25 on every strand collection

## Limitations

- Both strands and both paired-end files require separate Step 1/Step 2 runs, multiplying preprocessing and runtime
- LiME assumes reads' strand direction is unknown, so omitting reverse complements can misorient reads and degrade classification
- Classification quality is validated on simulated benchmarking data; real metagenome concordance was only assessed against MagicBlast

## Evidence

- [readme] since reads' strand direction is unknown, both the original sequences and their reverse complements should be considered to keep reads properly oriented and to improve their classification.: "since reads' strand direction is unknown, both the original sequences and their reverse complements should be considered to keep reads properly oriented and to improve their classification."
- [readme] For paired-end read collections, thus, the procedures at Step 1. and Step 2. have to be repeated for the data structures of both strands (forward and reverse complement) of both paired-end reads.: "For paired-end read collections, thus, the procedures at Step 1. and Step 2. have to be repeated for the data structures of both strands (forward and reverse complement) of both paired-end reads."
- [methods] In this second phase, we only consider ambiguous entries.: "In this second phase, we only consider ambiguous entries."
- [abstract] 99.9% of the positive control reads are correctly assigned and the percentage of classified reads of the negative control is less than 0.01%: "99.9% of the positive control reads are correctly assigned and the percentage of classified reads of the negative control is less than 0.01%"
