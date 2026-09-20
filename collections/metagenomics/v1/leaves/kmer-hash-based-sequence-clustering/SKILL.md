---
name: kmer-hash-based-sequence-clustering
description: Use when you need to reduce redundancy of very large protein sequence
  sets (e.
when_to_use_negative:
- When exact pairwise verification with the exact similarity measure is mandatory
  and falsely clustered pairs are costly — verification still requires quadratic time,
  defeating the linear-time purpose
- When neither input size N nor cluster count K is large enough that a near-quadratic
  algorithm (CD-HIT, UCLUST, kclust) would be impracticably slow
- When the items lack a set of m keys per item satisfying shared-key conditions, since
  Linclust requires such a hashing scheme
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: 'Primary linear-time clustering algorithm: hash-selected k-mers, Hamming-distance
    preclustering, ungapped alignment filter, gapped alignment verification, greedy
    incremental clustering'
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite that integrates Linclust ('mmseqs linclust' command) with --kmer-per-seq,
    --min-seq-id, --cov-mode and -c parameters
  repo: https://github.com/soedinglab/mmseqs2
- name: UCLUST
  role: Reference k-mer-prefilter clustering tool run with --id 0.5 to benchmark Linclust
    sensitivity and average cluster size
- name: CD-HIT 4.6
  role: Comparison clustering tool run with -n 3 -c 0.5 (also -n 4 -c 0.7, -n 5 -c
    0.9) in the benchmark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:4343dfa695ef8ae54818f60c4f744fd54f0b013c7c528fd6002fc413d75d02a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kmer-hash-based-sequence-clustering

## Summary

Clustering huge protein sequence sets in linear time using Linclust (MMseqs2), which selects m k-mers per sequence by lowest hash values, finds exact k-mer matches, and greedily clusters after Hamming-distance and ungapped-alignment prefiltering. Sensitivity can be tuned by increasing the number of selected k-mers per sequence (m) at a moderate runtime cost.

## When to use

Use when you need to reduce redundancy of very large protein sequence sets (e.g. UniProt releases, >1 billion metagenomic fragments) at a fixed sequence-identity threshold and linear O(N) runtime matters, or when a k-mer prefilter clustering at 50% identity under-clusters and you want to recover sensitivity by raising k-mers per sequence.

## When NOT to use

- When exact pairwise verification with the exact similarity measure is mandatory and falsely clustered pairs are costly — verification still requires quadratic time, defeating the linear-time purpose
- When neither input size N nor cluster count K is large enough that a near-quadratic algorithm (CD-HIT, UCLUST, kclust) would be impracticably slow
- When the items lack a set of m keys per item satisfying shared-key conditions, since Linclust requires such a hashing scheme

## Inputs

- Protein sequence set in FASTA format (e.g. UniProt 2016_03 subset)
- Identity/coverage thresholds (e.g. 50%, 70%, 90% sequence identity; coverage mode 1, coverage 0.9)
- Number of k-mers per sequence m (e.g. --kmer-per-seq 20 default, 80 for higher sensitivity)

## Outputs

- Cluster assignment (cluster TSV / rep-seq) mapping each sequence to a representative, choosing the longest sequence per cluster
- Summary table of average cluster sizes, runtimes, and sensitivity relative to a reference clustering (e.g. linclust_m80_comparison.csv)
- Runtime measurements for Linclust default (m=20), Linclust m=80, and reference tool (UCLUST)

## How to apply

Transform sequences into a reduced 13-letter alphabet, compute hash values for all k-mers, and select the m k-mers with the lowest hash values per sequence (default m=20). Generate the k-mer table, sort by k-mer index to find exact k-mer matches, choose the longest sequence per k-mer group as center, then filter candidates with a full-alphabet Hamming-distance preclustering followed by an ungapped local-alignment filter; survivors are verified with AVX2/SSE4.1-vectorized gapped local alignment and clustered greedily (longest sequence as representative). Run via 'mmseqs linclust' with identity/coverage thresholds such as --min-seq-id 0.5 --cov-mode 1 -c 0.9. If sensitivity is insufficient relative to a reference clustering (e.g. UCLUST --id 0.5, where Linclust yields 13% more clusters), raise --kmer-per-seq from 20 to 80, which lifts sensitivity to a UCLUST-comparable level at only ~1.5–2x longer runtime. Validate by comparing average cluster sizes against the reference clustering and confirming the runtime ratio stays moderate.

## Related tools

- **Linclust** (Primary linear-time clustering algorithm: hash-selected k-mers, Hamming-distance preclustering, ungapped alignment filter, gapped alignment verification, greedy incremental clustering) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite that integrates Linclust ('mmseqs linclust' command) with --kmer-per-seq, --min-seq-id, --cov-mode and -c parameters) — https://github.com/soedinglab/mmseqs2
- **UCLUST** (Reference k-mer-prefilter clustering tool run with --id 0.5 to benchmark Linclust sensitivity and average cluster size)
- **CD-HIT 4.6** (Comparison clustering tool run with -n 3 -c 0.5 (also -n 4 -c 0.7, -n 5 -c 0.9) in the benchmark)

## Examples

```
mmseqs linclust uniprot.fasta clusters tmp --kmer-per-seq 80 --min-seq-id 0.5 --cov-mode 1 -c 0.9
```

## Evaluation signals

- Average cluster size of Linclust m=80 clustering increases toward the UCLUST --id 0.5 reference (sensitivity recovered, consistent with Supplementary Fig. S4 trend)
- Runtime ratio of Linclust m=80 over default m=20 remains moderate (approximately 1.5–2x)
- Cluster representatives are the longest sequences in their clusters (greedy incremental clustering invariant)
- All reported clusters satisfy the exact distance measure (sequence identity and coverage thresholds, e.g. --min-seq-id 0.5 --cov-mode 1 -c 0.9) because edges are verified
- Runtime scales approximately linearly with input set size N rather than N x K

## Limitations

- At m=20 k-mers per sequence, sensitivity at 50% identity is lower than UCLUST (13% more clusters); sensitivity recovery requires m=80 with 1.5–2x runtime overhead
- Like the k-mer prefilter in CD-HIT, UCLUST, kclust, and MMseqs, the prefilter is only approximate; verification of edges with the exact similarity measure still requires quadratic time complexity
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting generalizability
- Traditional clustering approaches remain impracticably slow when both N and K are in the tens of millions; Linclust is designed for the huge-N regime

## Evidence

- [results] Increasing the number of selected k-mers per sequence from m=20 to m=80 raises Linclust's sensitivity at 50% identity to a level similar to UCLUST, at only 1.5 to 2 times longer runtime.: [evidence span withheld — anchor status: not_found]
- [methods] computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values: "computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values"
- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] We transform the sequence set into a reduced alphabet of 13 letters ... For each sequence, we extract m k-mers: "We transform the sequence set into a reduced alphabet of 13 letters"
- [discussion] Linclust's linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure.: "Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
