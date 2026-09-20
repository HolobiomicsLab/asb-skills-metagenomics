---
name: protein-kmer-hash-selection
description: Use when clustering huge protein sequence sets (e.g. UniProt-sized FASTA)
  and you need a per-sequence subset of k-mers that still guarantees that homologous
  sequences share at least one exact k-mer match, without enumerating all k-mers.
when_to_use_negative:
- Input sequences are very short, so that m=20 lowest-hash k-mers cannot be extracted
  or k between 10 and 14 is not meaningful
- The clustering criterion requires gapped-alignable homology at low identity where
  sharing an exact reduced-alphabet k-mer match is too insensitive and full all-against-all
  search (e.g. MMseqs2 search) is needed
- The data are non-sequence items for which no shared-key function can be defined,
  since Linclust requires m keys per item satisfying shared-key conditions
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Implements k-mer hash selection as Step 1 of its five-step linear-time clustering
    pipeline (reduced 13-letter alphabet, k=10-14, m=20 lowest-hash k-mers per sequence)
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite in which Linclust (including the k-mer table generation and
    in-place sorting via the OpenMP template library) is integrated and distributed
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:59ccd768434528af48af01075733733fa8ca350dc1b64d0a86de77024da9e964
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
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# protein-kmer-hash-selection

## Summary

Selects a fixed number m of representative k-mers per protein sequence by keeping those with the lowest 16-bit rolling hash values, the core k-mer generation step (Step 1) of Linclust's linear-time clustering algorithm, implemented in MMseqs2.

## When to use

Use when clustering huge protein sequence sets (e.g. UniProt-sized FASTA) and you need a per-sequence subset of k-mers that still guarantees that homologous sequences share at least one exact k-mer match, without enumerating all k-mers.

## When NOT to use

- Input sequences are very short, so that m=20 lowest-hash k-mers cannot be extracted or k between 10 and 14 is not meaningful
- The clustering criterion requires gapped-alignable homology at low identity where sharing an exact reduced-alphabet k-mer match is too insensitive and full all-against-all search (e.g. MMseqs2 search) is needed
- The data are non-sequence items for which no shared-key function can be defined, since Linclust requires m keys per item satisfying shared-key conditions

## Inputs

- Protein sequence set in FASTA format (e.g. UniProt 2016_03 release)
- Target sequence identity threshold (e.g. 0.9) and coverage mode parameters for choosing k

## Outputs

- Table of m selected k-mers per sequence (m = 20 lines per sequence) with k-mer index, sequence identifier, and diagonal position
- k-mer groups after sorting by k-mer index, with the longest sequence per group designated as center

## How to apply

Transform the protein sequence set into a reduced alphabet of 13 letters, then choose k adaptively between 10 and 14 using the specificity rule k = max{k_spec, k_seqid} so that random shared k-mers are unlikely at the target sequence identity. For each sequence, compute 16-bit rolling hash values for all its k-mers and retain only the m = 20 k-mers with the lowest hash values; this selection is deterministic and order-independent, so two homologous sequences sharing any k-mer will very likely have both retain it. Emit an mN-line table of (k-mer index, sequence id, diagonal position) that is subsequently sorted by k-mer index with the OpenMP template library in-place sort to form k-mer groups, each anchored by its longest (center) sequence. The rationale is that limiting stored k-mers to m per sequence bounds memory and runtime at O(mN), independent of the number of clusters K, unlike all-against-all k-mer matching which scales almost quadratically.

## Related tools

- **Linclust** (Implements k-mer hash selection as Step 1 of its five-step linear-time clustering pipeline (reduced 13-letter alphabet, k=10-14, m=20 lowest-hash k-mers per sequence)) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite in which Linclust (including the k-mer table generation and in-place sorting via the OpenMP template library) is integrated and distributed) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust uniprot.fasta clusterRes tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- The generated table contains exactly m = 20 k-mer entries per input sequence (mN lines total for N sequences)
- Every stored k-mer index was computed in the reduced 13-letter alphabet with k chosen via k = max{k_spec, k_seqid} in the range 10-14
- Each retained k-mer has one of the m lowest 16-bit rolling hash values of its sequence (selection is deterministic across runs)
- After sorting by k-mer index, every k-mer group contains a single designated center (the longest sequence), and re-sorting by center identifier groups all members with their center
- Downstream validation: every input sequence is assigned to exactly one representative and member-to-representative edges satisfy the chosen identity (e.g. 0.9) and coverage (0.9) thresholds

## Limitations

- The k-mer prefilter is approximate: sequence pairs homologous enough for clustering but sharing none of their m lowest-hash k-mers are missed (the filter thresholds are empirically set for a ~1% false negative rate)
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting its generalizability to other data types
- Verification of candidate edges uses the exact distance measure; guaranteeing exact results for all pairs would still require quadratic time

## Evidence

- [methods] We transform the sequence set into a reduced alphabet of 13 letters ... For each sequence, we extract m k-mers: [evidence span withheld — anchor status: not_found]
- [methods] computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values: "computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values"
- [methods] We sort this table by the k-mer index using the in-place sort from the OpenMP template library: "We sort this table by the k-mer index using the in-place sort from the OpenMP template library"
- [abstract] Linclust, the first clustering algorithm whose runtime scales as N, independent of K.: "We developed Linclust, the first clustering algorithm whose runtime scales as N, independent of K."
- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
