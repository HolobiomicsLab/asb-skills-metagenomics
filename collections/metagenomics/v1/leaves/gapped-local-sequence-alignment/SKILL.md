---
name: gapped-local-sequence-alignment
description: Use when clustering protein sequence sets at thresholds like --min-seq-id
  0.9 --cov-mode 1 -c 0.9 and survivors of the k-mer, Hamming, and ungapped filters
  must be verified with exact gapped local alignments before greedy incremental clustering.
when_to_use_negative:
- When sequences are expected to share similarity only outside any of the m selected
  lowest-hash k-mers, since Linclust aligns only center–member pairs sharing a k-mer
  and may miss such links
- When input is already a non-redundant sequence set or a precomputed clustering result
  — rerunning gapped alignments adds cost without benefit
- When exact all-against-all pairwise similarities beyond shared-k-mer pairs are required;
  Linclust's linear O(mN) time applies only to center-restricted comparisons
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: MMseqs2
  role: Provides the AVX2/SSE4.1-vectorized gapped local alignment module (Blosum62
    with compositional bias correction) used in step 4, and hosts the integrated linclust
    command
  repo: https://github.com/soedinglab/mmseqs2
- name: Linclust
  role: The linear-time clustering algorithm whose step 4 performs the gapped local
    alignment of filter survivors to their center sequences
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:5c070e80b3b0e0d01f6c136cb5d7b673946518e8bf50ec276fcbf5e4ba3cb142
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

# gapped-local-sequence-alignment

## Summary

Align candidate sequence pairs to their center sequences with an AVX2/SSE4.1-vectorized gapped local alignment module (Blosum62 with compositional bias correction), linking pairs that meet identity/coverage/E-value criteria. This is step 4 of the Linclust linear-time clustering pipeline, after Hamming-distance and ungapped-alignment filters have reduced the candidate set.

## When to use

Use when clustering protein sequence sets at thresholds like --min-seq-id 0.9 --cov-mode 1 -c 0.9 and survivors of the k-mer, Hamming, and ungapped filters must be verified with exact gapped local alignments before greedy incremental clustering.

## When NOT to use

- When sequences are expected to share similarity only outside any of the m selected lowest-hash k-mers, since Linclust aligns only center–member pairs sharing a k-mer and may miss such links
- When input is already a non-redundant sequence set or a precomputed clustering result — rerunning gapped alignments adds cost without benefit
- When exact all-against-all pairwise similarities beyond shared-k-mer pairs are required; Linclust's linear O(mN) time applies only to center-restricted comparisons

## Inputs

- Protein sequence set in FASTA format (e.g. UniProt 2016_03)
- Surviving center–member pairs from the Hamming-distance and ungapped-alignment filters (with stored diagonals)
- Clustering criteria: sequence identity threshold, coverage mode, coverage, and E-value threshold

## Outputs

- Linked center–member edges passing identity, coverage, and E-value criteria
- Final clustering in MMseqs2 clustering-result format (file 'clusterRes')

## How to apply

After Linclust steps 1–3 (k-mer table generation, k-mer grouping with center selection, and Hamming distance plus ungapped Blosum62 filtering), align each surviving sequence to its center sequence using the AVX2/SSE4.1-vectorized MMseqs2 gapped local alignment module with the Blosum62 substitution matrix and compositional bias correction. Link center–member pairs as edges when they satisfy the chosen criteria, e.g. --min-seq-id 0.9 --cov-mode 1 -c 0.9, with a default E-value threshold of 1e-3. Feed the resulting edges into step 5, greedy incremental clustering: add reverse edges, sort sequences by decreasing length, and iteratively peel off each top sequence with its still-unassigned neighbors to form clusters. Because only edges between center and member sequences sharing a k-mer are aligned, the total work remains O(mN) rather than quadratic in the set size N.

## Related tools

- **MMseqs2** (Provides the AVX2/SSE4.1-vectorized gapped local alignment module (Blosum62 with compositional bias correction) used in step 4, and hosts the integrated linclust command) — https://github.com/soedinglab/mmseqs2
- **Linclust** (The linear-time clustering algorithm whose step 4 performs the gapped local alignment of filter survivors to their center sequences) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust db db_cluster tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- Every input sequence is assigned to exactly one cluster representative in the output 'clusterRes' file
- All member-to-representative edges satisfy the chosen sequence identity (e.g. 0.9) and coverage (0.9) thresholds, matching the Linclust benchmark settings
- Cluster quality (e.g. functional consistency via GO annotations, local consistency via Pfam) is comparable to slower tools such as CD-HIT at 90%, 70%, and 50% identity levels on UniProt 2016_03
- Runtime scales linearly with input size N rather than N×K — spot-check that doubling the sequence set roughly doubles runtime
- Removed candidate pairs at the filter stages respect the empirically determined ungapped-alignment thresholds giving a 1% false negative rate

## Limitations

- Linclust clusters via shared k-mer keys and is approximate in scope: pairs that share none of the m selected k-mers are never aligned
- If falsely clustered pairs are costly, pairwise links need verification with the exact similarity measure, which for unrestricted comparison would still require quadratic time
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting its generalizability to other data types

## Evidence

- [methods] Sequences that pass the ungapped alignment filter are aligned to their center sequence using the AVX2/SSE4.1-vectorized alignment module: "Sequences that pass the ungapped alignment filter are aligned to their center sequence using the AVX2/SSE4.1-vectorized alignment module"
- [methods] Linclust and Linclust/MMseqs2 used the commands mmseqs linclust: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [discussion] We perform the clustering in step 5 of Fig. 1 with the greedy incremental clustering, because it always chooses the longest sequence as the cluster representative.: "We perform the clustering in step 5 of Fig. 1 with the greedy incremental clustering, because it always chooses the longest sequence as the cluster representative."
- [readme] MMseqs2 is a software suite to search and cluster huge protein and nucleotide sequence sets.: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
- [intro] We demonstrate that it produces clusterings of comparable quality as other tools that are orders of magnitude slower: "We demonstrate that it produces clusterings of comparable quality as other tools that are orders of magnitude slower and that it can cluster over a billion sequences within hours on a single server."
