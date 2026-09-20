---
name: hamming-distance-sequence-preclustering
description: Use when clustering large protein sequence sets (e.g., UniProt-scale
  FASTA) with Linclust/MMseqs2 and needing a fast, SIMD-vectorized exact-distance
  filter to prune center–member pairs after exact k-mer matching but before gapped
  alignment.
when_to_use_negative:
- When pairwise links must be verified with a measure Hamming distance along diagonals
  cannot capture, e.g., gapped similarities not detected by any exact k-mer match
  — no diagonal is stored and the pair is never compared
- When clustering items without definable keys — Linclust requires a set of m keys
  per item such that clustered items likely share at least one key
- When only a handful of sequences need clustering; simpler exact all-against-all
  tools avoid the pipeline's false-negative approximation
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Implements the Hamming distance pre-clustering as step 3A within its linear-time
    five-step clustering algorithm; invoked through the MMseqs2 linclust module
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite in which Linclust (including the SIMD-vectorized Hamming distance
    filter) is integrated; provides the linclust command with --min-seq-id, --cov-mode
    and -c parameters
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:4f5160970b88ac186804b3c4f2da6726951457e71f0416d28e29853e39aec669
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - Linclust
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Hamming distance sequence preclustering

## Summary

Step 3A of the Linclust five-step pipeline: compute Hamming distances in the full amino acid alphabet between each sequence and its k-mer group's center along stored diagonals, to cheaply remove matches that already satisfy sequence identity and coverage thresholds before slower alignment steps.

## When to use

Use when clustering large protein sequence sets (e.g., UniProt-scale FASTA) with Linclust/MMseqs2 and needing a fast, SIMD-vectorized exact-distance filter to prune center–member pairs after exact k-mer matching but before gapped alignment.

## When NOT to use

- When pairwise links must be verified with a measure Hamming distance along diagonals cannot capture, e.g., gapped similarities not detected by any exact k-mer match — no diagonal is stored and the pair is never compared
- When clustering items without definable keys — Linclust requires a set of m keys per item such that clustered items likely share at least one key
- When only a handful of sequences need clustering; simpler exact all-against-all tools avoid the pipeline's false-negative approximation

## Inputs

- Sorted k-mer table (k-mer index, sequence id, diagonal i-j) from Linclust step 2
- Protein sequence set in FASTA format (e.g., UniProt 2016_03)
- Target sequence identity threshold (e.g., --min-seq-id 0.9) and coverage mode/threshold (e.g., --cov-mode 1 -c 0.9)

## Outputs

- Reduced set of center–member candidate pairs, excluding those meeting identity and coverage on the full diagonal
- Pairs forwarded to the step 3B ungapped Blosum62 alignment filter

## How to apply

After step 2 has formed k-mer groups (reduced 13-letter alphabet, k between 10 and 14 chosen via the specificity rule k = max{k_spec, k_seqid}, m = 20 lowest-rolling-hash k-mers per sequence, longest sequence per group as center), compute the Hamming distance (number of mismatches) in the full amino acid alphabet between center and member along the stored diagonal i-j, using the AVX2/SSE4.1-vectorized implementation. If a member already satisfies the specified sequence identity and coverage thresholds over the entire diagonal, remove it from the results passed to step 3B; this exploits the fact that an exact diagonal match guarantees the identity criterion without any alignment. Rationale: this exact-measure verification is included in Linclust's O(mN) linear time complexity, unlike canopy-style prefilters whose exact verification is quadratic. Surviving pairs then face the ungapped Blosum62 alignment filter (step 3B) tuned to a 1% false negative rate before the gapped MMseqs2 alignment.

## Related tools

- **Linclust** (Implements the Hamming distance pre-clustering as step 3A within its linear-time five-step clustering algorithm; invoked through the MMseqs2 linclust module) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite in which Linclust (including the SIMD-vectorized Hamming distance filter) is integrated; provides the linclust command with --min-seq-id, --cov-mode and -c parameters) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust input.fasta clusterRes tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- Every pair removed at step 3A satisfies both the chosen sequence identity (e.g., 0.9) and coverage (e.g., 0.9) thresholds over the full diagonal — compute identities in the full amino acid alphabet to confirm
- Pairs surviving the filter do not exceed the ~1% false negative rate benchmarked for the overall three-step filtering cascade
- Runtime of the filtering stage scales linearly with input size N (O(mN) with m = 20 k-mers per sequence), not quadratically
- Final clustering assigns every input sequence to exactly one representative and all member-to-representative edges meet the identity and coverage criteria
- Reduction in candidate pairs passed to the gapped alignment step is large, consistent with the article's reported prefilter speedups

## Limitations

- Approximate: sequences not sharing any of the m lowest-hash k-mers with a cluster member are never aligned, so some true cluster edges are missed
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting generalizability to arbitrary item types
- The filter is tied to exact diagonal matches; it cannot rescue similarities requiring gaps that destroy the shared diagonal context

## Evidence

- [methods] For each k-mer group we compute the Hamming distance (the number of mismatches) in the full amino acid alphabet: "For each k-mer group we compute the Hamming distance (the number of mismatches) in the full amino acid alphabet"
- [methods] Members already meeting thresholds removed before 3b: "Members that already satisfy the specified sequence identity and coverage thresholds on the entire diagonal are removed from the results passed to step 3b"
- [discussion] Linear time includes exact verification: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [methods] Three consecutively slower filtering steps: [evidence span withheld — anchor status: not_found]
- [methods] MMseqs2 linclust command with clustering thresholds: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
