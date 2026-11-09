---
name: paired-end-similarity-matrix-computation
description: Use when classifying paired-end metagenomic reads with LiME and you must
  build similarity matrices for all four collections (Reads1F, Reads1RC, Reads2F,
  Reads2RC) merged against the reference genomes.
when_to_use_negative:
- The input requires a taxonomy classification decision (species/genus/phylum assignment)
  — that is handled by the separate Classify step, not the similarity computation
- The reads are single-end only and no paired-end strand re-examination is needed
  — only forward and reverse-complement structures of one mate are required
- The goal is alignment-based classification against a full nucleotide database (e.g.,
  BLAST-based filtering at 98% identity) — LiME's similarity matrix is alignment-free
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Framework whose ClusterLCP and ClusterBWT_DA components detect alpha-clusters
    and compute read-genome similarity values from ebwt/lcp/da structures
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds ebwt, lcp, and da data structures for the read and genome collections
    (can be used incrementally)
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: Alternative external-memory tool producing the generalized enhanced suffix
    array (gesa) converted via EGSAtoBCR into ebwt, lcp, and da files
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the ebwt data structures of read set R and genome set G with --lcp
    and --da options (optionally --trlcp k) to obtain the merged collection S structures
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:e3dd1568d4e69eebbccd6362e73cae0eb21a9e0566e8a7a83ee2c44296aa4100
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGAP
  - eGSA
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# paired-end-similarity-matrix-computation

## Summary

Computes a many-to-many read-vs-genome similarity matrix using LiME's alpha-cluster analysis over the eBWT, lcp, and da data structures of a merged read+genome collection, repeating the computation for forward and reverse-complement strands of both mates of paired-end reads.

## When to use

Use when classifying paired-end metagenomic reads with LiME and you must build similarity matrices for all four collections (Reads1F, Reads1RC, Reads2F, Reads2RC) merged against the reference genomes.

## When NOT to use

- The input requires a taxonomy classification decision (species/genus/phylum assignment) — that is handled by the separate Classify step, not the similarity computation
- The reads are single-end only and no paired-end strand re-examination is needed — only forward and reverse-complement structures of one mate are required
- The goal is alignment-based classification against a full nucleotide database (e.g., BLAST-based filtering at 98% identity) — LiME's similarity matrix is alignment-free

## Inputs

- Paired-end FASTA read files (e.g., setA2_1.fasta and setA2_2.fasta, 100 bps reads)
- Reference genome FASTA database (Refs.fasta, 930 genomes)
- ebwt, lcp, and da files for the read set R and genome set G
- Merged ebwt/lcp/da files for Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs collections

## Outputs

- Similarity values per read-genome pair from ClusterBWT_DA for each of the four strand/mate collections
- Input files for Classify containing similarity entries consumed for read assignment

## How to apply

Build ebwt, lcp, and da structures separately for the genome set G (930 genomes in Refs.fasta) and each read set, using BCR or eGSA, then merge read and genome structures with eGap setting both --lcp and --da (optionally --trlcp k). For paired-end data, generate reverse complements so four collections per dataset are merged: Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs. Run ClusterLCP with alpha=16 to detect alpha-clusters (blocks of ebwt(S) whose suffixes share a minimum context of length alpha), then ClusterBWT_DA with beta=0.25 to compute similarity values, using similarity approach (a) with EBWT=1 (reads both da and ebwt) or approach (b) with EBWT=0 (reads only da). The output is parsed downstream by Classify, which consumes the similarity files and assigns each read pair a status label C/U/A/H. The rationale is that LiME runs a many-to-many pairwise comparison producing a similarity matrix without ad-hoc in-memory reference database structures, only sequential scans.

## Related tools

- **LiME** (Framework whose ClusterLCP and ClusterBWT_DA components detect alpha-clusters and compute read-genome similarity values from ebwt/lcp/da structures) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds ebwt, lcp, and da data structures for the read and genome collections (can be used incrementally)) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (Alternative external-memory tool producing the generalized enhanced suffix array (gesa) converted via EGSAtoBCR into ebwt, lcp, and da files) — https://github.com/felipelouza/egsa
- **eGap** (Merges the ebwt data structures of read set R and genome set G with --lcp and --da options (optionally --trlcp k) to obtain the merged collection S structures) — https://github.com/felipelouza/egap

## Examples

```
ClusterLCP Reads1F+Refs.fasta numReads numRefs 16 threads; ClusterBWT_DA Reads1F+Refs.fasta 100 0.25 threads
```

## Evaluation signals

- All four merged collections per dataset (Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs) must be processed through ClusterLCP (alpha=16) and ClusterBWT_DA (beta=0.25)
- The lcp and da files must exist before running ClusterLCP; only the da file is needed if EBWT=0, both da and ebwt if EBWT=1
- Similarity output must contain entries for every read in R against genomes in G and be parseable by Classify into per-read C/U/A/H labels
- The number of alpha-clusters detected depends on the chosen alpha (minimum shared context length); alpha=16 should yield similarity values consistent with the paper's reported 99.9% of positive control reads correctly assigned
- The lcp values capped by --trlcp k must not degrade classification, since the used construction method does not affect the classification

## Limitations

- Both original read sequences and their reverse complements must be considered because reads' strand direction is unknown, quadrupling the collections for paired-end data
- eGap merging time can be long; the --trlcp k option trades lcp precision for speed by capping lcp values at k
- The tool reads data structures sequentially, so external memory usage is required and internal memory footprint should be bounded (scripts set INTMEM to 4096 MB for egsa and eGap)
- Reading only da (EBWT=0) versus both da and ebwt (EBWT=1) defines two distinct similarity approaches that yield different running times

## Evidence

- [methods] In Step 1, inspired by Remark 1, we build the collection Cα of α-clusters, which are blocks of symbols delimited by a pair of indices: [evidence span withheld — anchor status: not_found]
- [readme] ClusterLCP with alpha and threads parameters: "ClusterLCP with input parameters: name of the fasta file, total number of reads in S, total number of genomes in S, alpha and number of threads"
- [readme] both the original sequences and their reverse complements must be considered: "since reads' strand direction is unknown, both the original sequences and their reverse complements should be considered to keep reads properly oriented"
- [intro] LiME produces a similarity matrix via many-to-many pairwise comparison: "LiME also is the first metagenomic classifier that runs a many-to-many pairwise comparison and that is able to produce a similarity matrix"
- [methods] eGap merge with --lcp and --da options: [evidence span withheld — anchor status: not_found]
