---
name: ebwt-suffix-tree-simulation
description: Use when the input is only the eBWT file (reads.fasta.bwt) of raw reads
  and you need to detect eBWT clusters / local LCP minima for reference-free SNP/INDEL
  discovery under tight memory, without suffix-array structures.
when_to_use_negative:
- A reference genome and read-mapping pipeline are available and alignment-based variant
  calling is acceptable — this skill targets reference-free, assembly-free discovery
  on raw reads.
- You are running the old ebwt2snp (v1) pipeline, which explicitly requires the eBWT
  plus LCP and gSA arrays rather than on-the-fly LCP simulation.
- The read set contains 'N' symbols that have not been filtered or replaced — the
  tool requires fragments over A/C/G/T only.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: ebwt2InDel
  role: Implements the on-the-fly LCP extraction and suffix-tree simulation on the
    eBWT to detect SNP/INDEL clusters
  repo: github.com/nicolaprezza/ebwt2InDel
- name: BCR (BCR_LCP_GSA)
  role: Builds the multi-string eBWT (reads.fasta.bwt) of the read collection that
    is the sole input to the simulation
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGap
  role: Alternative external-memory tool for computing the BWT (and optionally LCP)
    of the sequence collection
  repo: https://github.com/felipelouza/egap
- name: egsa
  role: Alternative external-memory builder of the generalized suffix array and LCP
    array used by the older ebwt2snp design
  repo: https://github.com/felipelouza/egsa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:d78c44f71c9b420ef1bf5fbdd13b5e7744a96c8c9009c202920637411131e984
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BCR
  - BCR (BCR_LCP_GSA)
  - eGap
  - ebwt2InDel
  - egsa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ebwt-suffix-tree-simulation

## Summary

Simulates suffix-tree navigation directly on the extended Burrows-Wheeler Transform (eBWT) of a read collection by extracting LCP values on-the-fly, enabling ebwt2InDel's positional-clustering variant detection without materializing the LCP array or the generalized Suffix Array.

## When to use

Use when the input is only the eBWT file (reads.fasta.bwt) of raw reads and you need to detect eBWT clusters / local LCP minima for reference-free SNP/INDEL discovery under tight memory, without suffix-array structures.

## When NOT to use

- A reference genome and read-mapping pipeline are available and alignment-based variant calling is acceptable — this skill targets reference-free, assembly-free discovery on raw reads.
- You are running the old ebwt2snp (v1) pipeline, which explicitly requires the eBWT plus LCP and gSA arrays rather than on-the-fly LCP simulation.
- The read set contains 'N' symbols that have not been filtered or replaced — the tool requires fragments over A/C/G/T only.

## Inputs

- eBWT file of the input read collection (reads.fasta.bwt), built with BCR (BCR_LCP_GSA) or eGap
- FASTA/FASTQ read set previously reduced to symbols A/C/G/T (N-containing reads removed or replaced)

## Outputs

- Kmin, Kright, and localMin bitvectors computed in a single streaming pass over the eBWT
- Set of eBWT clusters (maximal intervals with no local LCP minimum with lcp >= kmin)
- Candidate SNP/INDEL variant positions with extracted left/right read contexts (via backward search and FL mapping)

## How to apply

Load the eBWT of the read collection previously built with BCR (BCR_LCP_GSA) or eGap, then simulate suffix-tree traversal using the technique that extracts the LCP on-the-fly from the eBWT. In a single streaming pass compute the Kmin, Kright, and localMin bitvectors without materializing the suffix array. Detect eBWT clusters as maximal eBWT intervals containing no local LCP minimum with lcp >= kmin, marking positions where read contexts diverge (candidate variant positions). This replaces the ebwt2snp v1 design that required a full scan of the eBWT plus LCP and gSA arrays; skipping those arrays cuts working space by roughly an order of magnitude (about 8x more space-efficient per the README), at a slight runtime cost.

## Related tools

- **ebwt2InDel** (Implements the on-the-fly LCP extraction and suffix-tree simulation on the eBWT to detect SNP/INDEL clusters) — github.com/nicolaprezza/ebwt2InDel
- **BCR (BCR_LCP_GSA)** (Builds the multi-string eBWT (reads.fasta.bwt) of the read collection that is the sole input to the simulation) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGap** (Alternative external-memory tool for computing the BWT (and optionally LCP) of the sequence collection) — https://github.com/felipelouza/egap
- **egsa** (Alternative external-memory builder of the generalized suffix array and LCP array used by the older ebwt2snp design) — https://github.com/felipelouza/egsa

## Examples

```
./ebwt2InDel -1 reads.fasta.bwt -o output.snp
```

## Evaluation signals

- No .lcp or generalized suffix array files are produced or loaded — the run operates solely on reads.fasta.bwt, with memory roughly 8x below the ebwt2snp v1 design.
- Kmin, Kright, and localMin bitvectors are computed in one streaming pass over the eBWT (single-scan invariant).
- Detected eBWT clusters are maximal intervals containing no local LCP minimum with lcp >= kmin; no cluster should contain such a minimum.
- Downstream output.snp exists, parses as SNP/INDEL records in KisSNP2-style fasta format, and retains events supportable by at least m reads (paper suggests m = 5 via filter_snp for coverages >= 25x).

## Limitations

- The BWT-only approach is slightly slower than ebwt2snp v1 despite being ~8 times more space-efficient.
- eBWT computation time (e.g., 23 hours 19 minutes with BCR on whole-genome data) is excluded from the tool's reported cost and must be budgeted separately.
- Input fragments must contain only A, C, G, T — reads containing 'N' must be filtered out or Ns replaced/removed beforehand.

## Evidence

- [intro] On-the-fly LCP replaces suffix array: "we overcome the above limitation by using a recent algorithmic result [56] that shows how to extract the LCPon-the-fly from the eBWT"
- [abstract] Order-of-magnitude working-space reduction: "perform the whole analysis using just the BWT, thus reducing the working space by one order of magnitude"
- [readme] v2 works directly on the input BWT: "The new version detect also InDels and works directly on the input BWT, without building the LCP and GSA arrays."
- [readme] Space-efficiency tradeoff of the new version: "This is about 8 times more space-efficient (but slightly slower) than version 1."
- [methods] eBWT must be prebuilt as input: "First, the BWT of the input dataset(s) must be built."
