---
name: alignment-free-read-genome-similarity-scoring
description: Use when you have ebwt/lcp/da files for a combined read+genome fasta
  and need an alignment-free similarity score of every read against every genome,
  or when re-implementing LiME's Step 2 scoring with IUPAC-aware symbol matching.
when_to_use_negative:
- The goal is alignment-based mapping or base-level alignment of reads to reference
  genomes (LiME is explicitly alignment-free and mapping-free)
- Only reads are available and no reference genome collection G (or its data structures)
  exists — the scoring is a read-versus-genome comparison
- The similarity measure must ignore IUPAC ambiguity codes while still scoring them
  — the EBWT=1 variant is required; the preliminary LightMetaEbwt_Alcob version treated
  ambiguity symbols as placeholders
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: LiME (ClusterBWT_DA)
  role: Executes Step 2, computing the r x g similarity matrices from the alpha-clusters;
    compiled with EBWT=1 (eBWT symbol-matching with IUPAC handling) or EBWT=0 (document-array
    color presence)
  repo: https://github.com/veronicaguerrini/LiME
- name: LiME (ClusterLCP)
  role: Executes Step 1, scanning da(S) and lcp(S) sequentially to detect the alpha-clusters
    that Step 2 scores
  repo: https://github.com/veronicaguerrini/LiME
- name: eGAP
  role: Merges the ebwts of the read set R and genome set G; use --lcp and --da (or
    --trlcp k) to also obtain the LCP and document arrays needed by ClusterLCP
  repo: https://github.com/felipelouza/egap
- name: BCR
  role: Builds ebwt, lcp and da for the collection S from a single fasta, or incrementally
    updates G's data structures with reads R
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGSA
  role: Alternative external-memory builder of the generalized suffix array, LCP array
    and BWT; output FILE.K.gesa is converted with EGSAtoBCR into the .ebwt/.lcp/.da
    files LiME consumes
  repo: https://github.com/felipelouza/egsa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:5d692c92449149cd90495ae5530d323821c248c62ff5a4f38a57361625cf70b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - LiME
  - LiME (ClusterBWT_DA)
  - LiME (ClusterLCP)
  - eGAP
  - eGSA
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# alignment-free-read-genome-similarity-scoring

## Summary

Compute an r x g similarity matrix between metagenomic reads and reference genomes by scanning the eBWT, LCP and document array of the combined collection S, using the alpha-clusters built by ClusterLCP (Step 2 of LiME's fixed three-step pipeline).

## When to use

Use when you have ebwt/lcp/da files for a combined read+genome fasta and need an alignment-free similarity score of every read against every genome, or when re-implementing LiME's Step 2 scoring with IUPAC-aware symbol matching.

## When NOT to use

- The goal is alignment-based mapping or base-level alignment of reads to reference genomes (LiME is explicitly alignment-free and mapping-free)
- Only reads are available and no reference genome collection G (or its data structures) exists — the scoring is a read-versus-genome comparison
- The similarity measure must ignore IUPAC ambiguity codes while still scoring them — the EBWT=1 variant is required; the preliminary LightMetaEbwt_Alcob version treated ambiguity symbols as placeholders

## Inputs

- Combined fasta of reads R and reference genomes G (e.g. ReadsF+Refs.fasta)
- eBWT file (fileFasta.ebwt) of the collection S — required when EBWT=1
- LCP array file (fileFasta.lcp) of the collection S
- Document array file (fileFasta.da) of the collection S
- Parameters: read length, alpha (context length, e.g. 16 for 100 bp paired-end reads), beta (threshold, e.g. 0.25), number of threads

## Outputs

- Similarity matrices of dimensions r x g (reads x genomes) for the forward and reverse-complement data structures of both mates
- ClusterBWT_DA output files consumed by Classify for read-to-taxon assignment

## How to apply

First obtain ebwt(S), lcp(S) and da(S) either from a single fasta with BCR or eGSA, or build them separately for reads R and genomes G and merge with eGAP using --lcp and --da. Run ClusterLCP sequentially over da(S) and lcp(S) with alpha=16 (for 100 bp paired-end reads) to detect alpha-clusters: blocks of ebwt(S) whose suffixes from both R and G share a context of length at least alpha. Then run ClusterBWT_DA — compiled with EBWT=1 for the eBWT variant that counts symbol matches within alpha-clusters while resolving IUPAC ambiguity codes (an ambiguity code matches itself or any of its constituent letters), or EBWT=0 for the document-array color-presence variant — passing the read length and classification threshold beta=0.25. Because strand direction is unknown, Step 1 and Step 2 are repeated on the forward and reverse-complement data structures of both mates, yielding four r x g similarity matrices that feed Classify. The classification itself does not depend on which construction algorithm produced ebwt, da and lcp.

## Related tools

- **LiME (ClusterBWT_DA)** (Executes Step 2, computing the r x g similarity matrices from the alpha-clusters; compiled with EBWT=1 (eBWT symbol-matching with IUPAC handling) or EBWT=0 (document-array color presence)) — https://github.com/veronicaguerrini/LiME
- **LiME (ClusterLCP)** (Executes Step 1, scanning da(S) and lcp(S) sequentially to detect the alpha-clusters that Step 2 scores) — https://github.com/veronicaguerrini/LiME
- **eGAP** (Merges the ebwts of the read set R and genome set G; use --lcp and --da (or --trlcp k) to also obtain the LCP and document arrays needed by ClusterLCP) — https://github.com/felipelouza/egap
- **BCR** (Builds ebwt, lcp and da for the collection S from a single fasta, or incrementally updates G's data structures with reads R) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGSA** (Alternative external-memory builder of the generalized suffix array, LCP array and BWT; output FILE.K.gesa is converted with EGSAtoBCR into the .ebwt/.lcp/.da files LiME consumes) — https://github.com/felipelouza/egsa

## Examples

```
ClusterLCP ReadsF+Refs.fasta numReads numRefs 16 8; ClusterBWT_DA ReadsF+Refs.fasta 100 0.25 8
```

## Evaluation signals

- ClusterLCP output contains only alpha-clusters whose ebwt(S) symbols belong to both reads R and genomes G with a shared context of length >= alpha
- ClusterBWT_DA produces one r x g similarity matrix per strand per mate (four matrices for paired-end input)
- All similarity values fall in [0,1); entries for reads whose maximum similarity is below beta=0.25 must not be reported as classified (C) by Classify
- Scores respect IUPAC semantics under EBWT=1: an ambiguity code matches itself or any letter associated with its code
- Results are identical regardless of whether ebwt/lcp/da were built with BCR, eGSA or built separately and merged with eGAP

## Limitations

- The context length alpha must be tuned to read length (alpha=16 is recommended for 100 bp paired-end reads); finding the best data-structure construction method is explicitly out of scope
- Both forward and reverse-complement data structures must be processed since read strand direction is unknown, roughly doubling the work
- The IUPAC-aware symbol-matching variant requires EBWT=1 and the ebwt file; the EBWT=0 variant needs only the document array but does not count symbol matches
- eGap's --trlcp k option caps all lcp values at k, which caps the effective shared context length available for alpha-cluster detection

## Evidence

- [methods] alpha-cluster detection feeding scoring: [evidence span withheld — anchor status: not_found]
- [methods] threshold semantics: [evidence span withheld — anchor status: not_found]
- [readme] two scoring approaches: "We analyze α-clusters in order to evaluate a degree of similarity between any read and any genome in *S* by using two different approaches"
- [readme] compile-time variant selection: "for the first approach (a) set EBWT=1 (default)"
- [full_text] IUPAC-aware matching in the eBWT variant: [evidence span withheld — anchor status: not_found]
