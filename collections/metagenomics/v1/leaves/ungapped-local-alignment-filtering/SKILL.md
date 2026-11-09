---
name: ungapped-local-alignment-filtering
description: Use when implementing a linear-time sequence clustering pipeline and
  needing to prune center-vs-member candidate pairs after Hamming pre-clustering but
  before expensive gapped alignment, to cut gapped-alignment workload while keeping
  ~1% false negatives.
when_to_use_negative:
- Pairs already removed by the Hamming distance pre-clustering step (3a) because they
  meet the identity and coverage thresholds on the entire diagonal — the ungapped
  filter would be redundant
- Analyses that require full gapped sensitivity on every k-mer-sharing pair, since
  this filter is approximate and tuned for a ~1% false negative rate, not exact
- Datasets without an assigned center sequence and stored diagonal (i.e. no prior
  k-mer grouping from step 2), as the filter aligns along the precomputed diagonal
  only
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Implements the ungapped local alignment filter as step 3b of its five-step
    linear-time clustering algorithm, using AVX2/SSE4.1 vectorized Hamming and ungapped
    alignment computations
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite hosting Linclust; after the ungapped filter, its vectorized
    gapped local alignment module (step 4) processes the surviving pairs
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:1166cd0f371404f00bed66dfb17c24ae799051db8575af5bb13afc75f84cc3c7
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
yes_rate: 1.0
n_claims: 3
---

# ungapped-local-alignment-filtering

## Summary

A fast intermediate filter in Linclust's clustering pipeline (step 3b) that computes optimal ungapped local alignment scores between each k-mer group's center sequence and its members along the stored diagonal, discarding matches whose score-per-diagonal-length falls below empirically tuned thresholds, so that only surviving pairs enter the costly gapped alignment step.

## When to use

Use when implementing a linear-time sequence clustering pipeline and needing to prune center-vs-member candidate pairs after Hamming pre-clustering but before expensive gapped alignment, to cut gapped-alignment workload while keeping ~1% false negatives.

## When NOT to use

- Pairs already removed by the Hamming distance pre-clustering step (3a) because they meet the identity and coverage thresholds on the entire diagonal — the ungapped filter would be redundant
- Analyses that require full gapped sensitivity on every k-mer-sharing pair, since this filter is approximate and tuned for a ~1% false negative rate, not exact
- Datasets without an assigned center sequence and stored diagonal (i.e. no prior k-mer grouping from step 2), as the filter aligns along the precomputed diagonal only

## Inputs

- Sorted k-mer group table with center identifiers and diagonals (i-j) from step 2
- Protein sequences from the input FASTA set (e.g. UniProt 2016_03)
- Sequence identity and coverage thresholds (e.g. --min-seq-id 0.9, -c 0.9 with --cov-mode 1)
- Empirically determined ungapped score thresholds calibrated for a 1% false negative rate

## Outputs

- Filtered list of center-member candidate pairs that pass the ungapped alignment score threshold
- Removed pairs (members already satisfying identity/coverage on the diagonal, or failing the ungapped score filter)

## How to apply

After Hamming distance pre-clustering (step 3a) has removed members already meeting the sequence identity and coverage thresholds on the full diagonal, compute for each remaining center-member pair the optimal ungapped local alignment in the full amino acid alphabet along the stored diagonal (position i-j) using AVX2/SSE4.1 vectorization. Score the alignment with Blosum62-derived scores and filter out matches where the ungapped alignment score divided by the length of the diagonal is below thresholds that were empirically calibrated to yield a 1% false negative rate. This step is the middle of Linclust's three consecutively slower and more sensitive steps, ensuring the overall complexity stays O(mN) while gapped alignment (step 4, MMseqs2 module with Blosum62 and compositional bias correction) is applied only to surviving pairs.

## Related tools

- **Linclust** (Implements the ungapped local alignment filter as step 3b of its five-step linear-time clustering algorithm, using AVX2/SSE4.1 vectorized Hamming and ungapped alignment computations) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite hosting Linclust; after the ungapped filter, its vectorized gapped local alignment module (step 4) processes the surviving pairs) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust UniProt.fasta clusterRes tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- The filter's thresholds are calibrated such that the false negative rate on benchmark edges is ~1%
- Pairs removed by this filter should have ungapped alignment score / diagonal length below the calibrated threshold; pairs passed should proceed to gapped alignment
- Any member already satisfying the target identity (e.g. 0.9) and coverage (0.9) thresholds on the full diagonal in step 3a must NOT reach this filter
- Final clustering must still satisfy the chosen thresholds: every member-to-representative edge passes --min-seq-id 0.9 -c 0.9 --cov-mode 1 validation
- End-to-end runtime of the pipeline should remain linear, O(mN), with this filter demonstrably reducing the number of pairs entering the gapped alignment module

## Limitations

- The filter is approximate: thresholds tuned for a 1% false negative rate mean ~1% of true cluster edges may be lost before gapped alignment
- Only the precomputed diagonal (i-j) between center and member is evaluated; gapped or shifted alignments are not considered at this stage
- As a k-mer-key-based prefilter, Linclust assumes clustered pairs share at least one of the m lowest-hash k-mers; pairs violating this shared-key assumption are never seen by this filter

## Evidence

- [methods] Step 3B: Ungapped alignment filtering: "For each k-mer group we compute the optimal ungapped, local alignment between the center sequence and each sequence"
- [methods] Ungapped filter criterion: "We filter out matches between center and member sequences if the ungapped alignment score divided by the length of the diagonal is very low."
- [methods] Removal by Hamming pre-clustering: "Members that already satisfy the specified sequence identity and coverage thresholds on the entire diagonal are removed from the results passed to step 3b"
- [discussion] Linear complexity includes exact verification: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [full_text] Three consecutively slower steps: [evidence span withheld — anchor status: not_found]
