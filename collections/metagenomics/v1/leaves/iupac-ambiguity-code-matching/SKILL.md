---
name: iupac-ambiguity-code-matching
description: Use when computing read-to-genome similarity with LiME's eBWT-based symbol-matching
  strategy and reference genomes or reads contain IUPAC ambiguity codes that must
  contribute matches rather than being discarded.
when_to_use_negative:
- If you only have the document array and prefer the color-presence variant of Step
  2, compile with EBWT=0 instead — that approach reads only da(S) and does not perform
  eBWT symbol matching
- If the goal is alignment-based read mapping with identity thresholds (e.g. BLAST
  at 98% sequence identity), LiME's alignment-free similarity measure is the wrong
  tool
- If reads need no ambiguity-code handling and a lightweight scan of only da(S) suffices,
  the document-array variant avoids requiring fileFasta.ebwt
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Provides ClusterBWT_DA, compiled with EBWT=1, to compute the read-genome similarity
    matrices using eBWT symbol matching with IUPAC ambiguity-code awareness
  repo: https://github.com/veronicaguerrini/LiME
- name: ClusterLCP
  role: Upstream step that detects the alpha-clusters of ebwt(S) symbols from reads
    and genomes sharing a minimum context of length alpha, within which matches are
    counted
  repo: https://github.com/veronicaguerrini/LiME
- name: eGap
  role: Merges the ebwts of read set R and genome set G, computing lcp and da via
    the --lcp and --da options for the collection S scanned by ClusterLCP/ClusterBWT_DA
  repo: https://github.com/felipelouza/egap
- name: BCR
  role: Alternative tool to build ebwt, lcp, and da from a single fasta of reads plus
    genomes before similarity scoring
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:8828bb6d40f321cf62fd38505c36d31593b665f654bb88dbe056320b05adc3fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - ClusterLCP
  - LiME
  - eGAP
  - eGap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# iupac-ambiguity-code-matching

## Summary

LiME's first similarity-measure variant (ClusterBWT_DA compiled with EBWT=1) counts symbol matches between each metagenomic read and each reference genome within alpha-clusters, treating IUPAC ambiguity codes as matching themselves or any concrete nucleotide letter associated with the code, instead of considering them placeholders.

## When to use

Use when computing read-to-genome similarity with LiME's eBWT-based symbol-matching strategy and reference genomes or reads contain IUPAC ambiguity codes that must contribute matches rather than being discarded.

## When NOT to use

- If you only have the document array and prefer the color-presence variant of Step 2, compile with EBWT=0 instead — that approach reads only da(S) and does not perform eBWT symbol matching
- If the goal is alignment-based read mapping with identity thresholds (e.g. BLAST at 98% sequence identity), LiME's alignment-free similarity measure is the wrong tool
- If reads need no ambiguity-code handling and a lightweight scan of only da(S) suffices, the document-array variant avoids requiring fileFasta.ebwt

## Inputs

- Merged ebwt(S), lcp(S), and da(S) data structures for the combined collection of reads and reference genomes (e.g. built with BCR/eGSA from a single fasta or merged with eGap using --lcp and --da)
- Alpha-clusters produced by ClusterLCP (context length alpha, e.g. 16 for 100 bp paired-end reads)
- Fasta file containing reads and reference genomes (e.g. ReadsF+Refs.fasta)
- Read length and classification threshold beta (e.g. 0.25)

## Outputs

- r x g similarity matrices (one entry per read-genome pair) for the forward and reverse-complement data structures of both mates, output by ClusterBWT_DA
- Similarity scores consumed by Classify, where entries with maxSim exceeding beta (e.g. 0.25) become classified (C)

## How to apply

Compile LiME with EBWT=1 (the default) so ClusterBWT_DA uses the eBWT symbol-matching variant that reads both da(S) and ebwt(S). After ClusterLCP has detected alpha-clusters (blocks of ebwt(S) containing symbols from both reads R and genomes G whose suffixes share a context of minimum length alpha, e.g. alpha=16 for 100 bp paired-end reads), ClusterBWT_DA counts, within each alpha-cluster, how many symbols match between each read and each genome, allowing an IUPAC ambiguity code to match itself or any letter associated with its code. Run ClusterBWT_DA per data structure with the read length, threshold beta=0.25, and a thread count to fill the r x g similarity matrices for the forward and reverse-complement structures of both mates. This variant requires both fileFasta.da and fileFasta.ebwt as input, whereas the EBWT=0 variant needs only fileFasta.da and relies on document-array color presence instead.

## Related tools

- **LiME** (Provides ClusterBWT_DA, compiled with EBWT=1, to compute the read-genome similarity matrices using eBWT symbol matching with IUPAC ambiguity-code awareness) — https://github.com/veronicaguerrini/LiME
- **ClusterLCP** (Upstream step that detects the alpha-clusters of ebwt(S) symbols from reads and genomes sharing a minimum context of length alpha, within which matches are counted) — https://github.com/veronicaguerrini/LiME
- **eGap** (Merges the ebwts of read set R and genome set G, computing lcp and da via the --lcp and --da options for the collection S scanned by ClusterLCP/ClusterBWT_DA) — https://github.com/felipelouza/egap
- **BCR** (Alternative tool to build ebwt, lcp, and da from a single fasta of reads plus genomes before similarity scoring) — https://github.com/giovannarosone/BCR_LCP_GSA

## Examples

```
make && ./ClusterBWT_DA ReadsF+Refs.fasta 100 0.25 4
```

## Evaluation signals

- ClusterBWT_DA runs only if both fileFasta.da and fileFasta.ebwt exist (required when EBWT=1); missing the .ebwt file indicates wrong build variant or preprocessing
- Output of the subsequent Classify step must contain lines in the comma-separated format C/U/A/H,IdSeqRead,TaxID,maxSim, with only read pairs whose maximum similarity exceeds beta reported as classified (C)
- Selected genomes must fall within tolerance 0.02 of the maximum similarity value for each read
- On simulated data the pipeline is expected to reach high precision and specificity: 99.9% of positive control reads correctly assigned and less than 0.01% of negative control reads classified
- Results should be invariant to the algorithm used to build ebwt(S), da(S), and lcp(S) (BCR, eGSA, or eGap merging)

## Limitations

- The previous eBWT-based strategy treated IUPAC symbols as placeholders, so ambiguity-code-aware matching is specific to this newer implementation and its EBWT=1 variant
- Both the original read sequences and their reverse complements must be processed (Step 1 and Step 2 repeated for forward and reverse-complement structures of both mates), doubling the work for paired-end data
- To find the best method for building the data structures is explicitly not an aim of the paper; classification performance depends on alpha=16 and beta=0.25 settings tuned for 100 bp paired-end reads
- The paper defers parameter sweep details (Table S2) to the supplementary material, so sensitivity to alpha and beta beyond the stated defaults is not documented in the main text

## Evidence

- [other] LiME counts how many symbols match between each read and each genome within any α-cluster, also taking into account the IUPAC ambiguity code, where an ambiguity code can match itself or any letter associated with its code.: [evidence span withheld — anchor status: not_found]
- [readme] Compile EBWT=1 for symbol-matching variant.: "One can choose between the two strategies of Step 2. by setting parameter EBWT"
- [methods] we detect α-clusters, i.e., the blocks of ebwt(S) containing symbols belonging both to R and to G: [evidence span withheld — anchor status: not_found]
- [intro] The previous strategy did not take into account the symbols in the International Union of Pure and Applied Chemistry (IUPAC)2 code [46], indeed they were considered as placeholders.: "The previous strategy did not take into account the symbols in the International Union of Pure and Applied Chemistry (IUPAC)2 code [46], indeed they were considered as placeholders."
- [readme] to run ClusterBWT_DA we need only fileFasta.da if EBWT=0, both fileFasta.da and fileFasta.ebwt if EBWT=1: "to run ClusterBWT_DA we need only fileFasta.da if EBWT=0, both fileFasta.da and fileFasta.ebwt if EBWT=1"
