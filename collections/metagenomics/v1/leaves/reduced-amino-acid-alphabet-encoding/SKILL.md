---
name: reduced-amino-acid-alphabet-encoding
description: Use when building a k-mer prefilter for protein sequence clustering or
  search where alphabet reduction plus lowest-hash k-mer selection must keep the index
  linear in input size N.
when_to_use_negative:
- Input sequences are nucleotide rather than protein, since the 13-letter reduced
  amino acid alphabet and Blosum62 scoring are protein-specific
- Exact all-against-all pairwise similarity is required rather than an approximate
  k-mer-based prefilter, as k-mer matching can miss sequence pairs sharing no selected
  k-mer
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: Implements the reduced-alphabet k-mer table generation as step 1 of the five-step
    linear-time clustering workflow, integrated into MMseqs2
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite in which Linclust (and its k-mer encoding step) is integrated;
    provides the OpenMP in-place sort and AVX2/SSE4.1-vectorized modules
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:707f16eb3ba5413f21b653965bc39b91dd6171ac65c16c741579911b6f92d833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - Linclust
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Reduced amino acid alphabet encoding

## Summary

Reduces protein sequences to a 13-letter alphabet and selects m=20 representative k-mers per sequence via a 16-bit rolling hash, as done in Linclust's step 1 to build a linear-size k-mer table for clustering huge protein sequence sets.

## When to use

Use when building a k-mer prefilter for protein sequence clustering or search where alphabet reduction plus lowest-hash k-mer selection must keep the index linear in input size N.

## When NOT to use

- Input sequences are nucleotide rather than protein, since the 13-letter reduced amino acid alphabet and Blosum62 scoring are protein-specific
- Exact all-against-all pairwise similarity is required rather than an approximate k-mer-based prefilter, as k-mer matching can miss sequence pairs sharing no selected k-mer

## Inputs

- Protein sequence set in FASTA format (e.g., UniProt 2016_03 release)
- Target sequence identity level (e.g., 0.9, 0.7, or 0.5) for choosing k

## Outputs

- k-mer table of mN lines sorted by k-mer index, with k-mer index, hash diagonal, sequence identifier, and center identifier columns

## How to apply

Reduce the amino acid alphabet of the input FASTA (e.g., UniProt 2016_03) to 13 letters, then choose k between 10 and 14 using the specificity rule k = max{k_spec, k_seqid}. For each sequence, compute 16-bit rolling hash values for all k-mers and extract the m = 20 k-mers per sequence with the lowest hash values. Sort the resulting mN-line table by k-mer index using the OpenMP template library in-place sort to form k-mer groups whose center is the longest sequence per group. This restricted k-mer extraction is what keeps Linclust's runtime at O(mN), independent of the number of clusters K.

## Related tools

- **Linclust** (Implements the reduced-alphabet k-mer table generation as step 1 of the five-step linear-time clustering workflow, integrated into MMseqs2) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite in which Linclust (and its k-mer encoding step) is integrated; provides the OpenMP in-place sort and AVX2/SSE4.1-vectorized modules) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust input.fasta clusterRes tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- Every input sequence contributes exactly m = 20 entries (or fewer for very short sequences) to the k-mer table
- The table row count is mN (or less), confirming linear scaling O(mN) with input set size N
- k-mer length k falls within the valid range 10–14 and satisfies k = max{k_spec, k_seqid} for the target identity
- Each entry's hash falls within the 16-bit range and the k-mer index column allows exact-match grouping after sorting
- Downstream clustering assigns every input sequence to exactly one representative satisfying the identity and coverage thresholds (e.g., 0.9 / 0.9)

## Limitations

- k-mer prefiltering is approximate: pairs of homologous sequences sharing none of the m selected lowest-hash k-mers are missed, yielding a nonzero false-negative rate (empirically tuned to ~1% at the ungapped alignment filter)
- Linclust requires that a set of m keys per item can be defined such that cluster members are likely to share at least one key, restricting generalizability
- Choosing too large a k for low identity thresholds (e.g., 50%) loses sensitivity; too small a k generates excessively large k-mer groups

## Evidence

- [methods] Reduced alphabet of 13 letters with lowest-hash k-mer selection: [evidence span withheld — anchor status: not_found]
- [methods] Selecting k-mers by lowest hash values: "computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values"
- [methods] In-place sorting by k-mer index to form k-mer groups: "We sort this table by the k-mer index using the in-place sort from the OpenMP template library"
- [abstract] Linclust is the first clustering algorithm with runtime scaling linearly as N: "We developed Linclust, the first clustering algorithm whose runtime scales as N, independent of K."
- [discussion] Longest sequence per k-mer group becomes the center: "Our rule in step 2 to choose the longest protein sequence per k-mer group as its center is well-suited to achieve large clusters"
