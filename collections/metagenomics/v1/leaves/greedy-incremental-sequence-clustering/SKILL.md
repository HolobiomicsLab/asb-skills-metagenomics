---
name: greedy-incremental-sequence-clustering
description: Use when you need to partition a large protein sequence set (e.g. UniProt-scale,
  hundreds of millions of sequences) into redundancy-reduced clusters after a k-mer
  prefilter and alignment steps have produced verified edges.
when_to_use_negative:
- The set of items lacks a definable set of m keys (e.g. k-mers) per item such that
  cluster members are likely to share at least one key — the article notes this shared-key
  condition restricts Linclust's generalizability
- Falsely clustered pairs are costly and every pairwise link must be verified with
  the exact similarity measure against all sequences — that regime still requires
  quadratic time, which this skill does not cover
- A clustering algorithm other than greedy-incremental is required, e.g. when representatives
  need not be the longest member or when non-overlapping assignment is enforced downstream
  before edge construction
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Implements the greedy incremental clustering as step 5 of its five-step pipeline,
    choosing the longest sequence as cluster representative
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite embedding Linclust; converts the greedy clustering result to
    its clustering-result format written to 'clusterRes'
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Alternative tool benchmarked on the same UniProt 2016_03 datasets for comparison
    of clustering quality and speed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:4c4dd211b240654186a8a379e46884e205d93c1bcfb9a73f9bdb395c7f132763
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# greedy-incremental-sequence-clustering

## Summary

Final clustering stage of the Linclust algorithm: after filtering and gapped alignment, sequences are added as reverse edges, sorted by decreasing length, and greedily peeled into clusters so that the longest sequence always becomes the cluster representative. This yields clustering in linear time O(mN) with representatives that maximize cluster stability.

## When to use

Use when you need to partition a large protein sequence set (e.g. UniProt-scale, hundreds of millions of sequences) into redundancy-reduced clusters after a k-mer prefilter and alignment steps have produced verified edges.

## When NOT to use

- The set of items lacks a definable set of m keys (e.g. k-mers) per item such that cluster members are likely to share at least one key — the article notes this shared-key condition restricts Linclust's generalizability
- Falsely clustered pairs are costly and every pairwise link must be verified with the exact similarity measure against all sequences — that regime still requires quadratic time, which this skill does not cover
- A clustering algorithm other than greedy-incremental is required, e.g. when representatives need not be the longest member or when non-overlapping assignment is enforced downstream before edge construction

## Inputs

- Protein sequence set in FASTA format (e.g. UniProt 2016_03 release)
- Verified alignment edges from Linclust steps 1–4 (center-to-member pairs passing Hamming, ungapped, and gapped-alignment filters)

## Outputs

- Final clustering in MMseqs2 clustering-result format (file 'clusterRes'), one cluster per line with representative sequence ID followed by member IDs

## How to apply

After Linclust steps 1–4 have linked pairs meeting the chosen criteria (e.g. --min-seq-id 0.9, --cov-mode 1, -c 0.9, E-value 1e-3), add reverse edges to the alignment result, sort all sequences by decreasing length, and iteratively peel off each top (longest) sequence together with all of its still-unassigned neighbors to form a new cluster; the peeled sequence becomes the representative. The rationale, per the paper, is that greedy incremental clustering 'always chooses the longest sequence as the cluster representative', and the step-2 rule of choosing the longest protein sequence per k-mer group as center is well-suited to achieve large clusters. Run this as step 5 of Linclust (mmseqs linclust), then write the final clustering in the MMseqs2 clustering-result format (file 'clusterRes'). Validate that every input sequence is assigned to exactly one representative and that each member-to-representative edge satisfies the chosen identity and coverage thresholds.

## Related tools

- **Linclust** (Implements the greedy incremental clustering as step 5 of its five-step pipeline, choosing the longest sequence as cluster representative) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite embedding Linclust; converts the greedy clustering result to its clustering-result format written to 'clusterRes') — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Alternative tool benchmarked on the same UniProt 2016_03 datasets for comparison of clustering quality and speed)

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9 seqDB seqDB_clust tmp
```

## Evaluation signals

- Every input sequence appears in exactly one cluster (assigned to exactly one representative) in the output 'clusterRes' file
- All member-to-representative edges satisfy the chosen sequence identity threshold (e.g. 0.9) and coverage threshold (0.9), matching the Linclust benchmark settings
- Each cluster's representative is the longest sequence in the cluster (the invariant of greedy incremental clustering)
- Output is valid MMseqs2 clustering-result format: first column = representative ID, second column = member ID
- Runtime scales linearly with input size N (O(mN)) rather than with the number of clusters K — verify timing growth is near-linear on scaling tests

## Limitations

- Edge links from the prefilter are approximate; however, unlike approximate canopy clustering, Linclust's O(mN) time includes verification of all edges with the exact distance measure
- Generalizable only to items for which a set of m keys per item can be defined such that cluster members are likely to share at least one key
- Datasets larger than available main memory are supported, but the algorithm is designed for the sequence-identity regime where shared k-mers are guaranteed by key choice; very low identity levels may need the more sensitive later steps

## Evidence

- [discussion] Greedy incremental clustering chooses the longest sequence as representative: "We perform the clustering in step 5 of Fig. 1 with the greedy incremental clustering, because it always chooses the longest sequence as the cluster representative."
- [discussion] Longest-per-group center rule aids large clusters: "Our rule in step 2 to choose the longest protein sequence per k-mer group as its center is well-suited to achieve large clusters"
- [discussion] Linear complexity includes exact edge verification: "Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [methods] Benchmark command with identity and coverage thresholds: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
