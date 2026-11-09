---
name: microbiome-body-site-clustering
description: Use when you have paired-end WGS or simulated long-read metagenome samples
  with known body-site labels and need to compare how k-mer-based tools cluster them
  by origin.
when_to_use_negative:
- Your goal is single-pair sample comparison rather than all-vs-all clustering across
  many samples — Libra/Mash/Simka matrices are designed for all-vs-all comparison
- Samples lack known body-site or origin labels, so clustering outcomes cannot be
  validated against expected groupings
- You need strain- or variant-level resolution rather than metagenome-level community
  composition/abundance comparison
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Libra
  role: Computes the all-vs-all k-mer abundance cosine-similarity score matrix in
    MAP mode with COSINESIMILARITY scoring and LOGARITHM weighting; runs on a Hadoop
    framework
  repo: https://github.com/iychoi/libra
- name: Mash
  role: Sketches each sample by unique k-mers and emits the pairwise MinHash distance
    matrix for comparison with Libra
- name: Simka
  role: Computes Jaccard-ab and Bray-Curtis distance matrices over the same read set
    as comparator metrics
- name: Hadoop
  role: Scalable MapReduce framework underlying Libra's preprocessing and scoring
    (hadoop jar libra-all.jar)
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:ee1c328484e8156348db95c48ab468c33508da3b93b12bb9a398066358224535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# microbiome-body-site-clustering

## Summary

Clusters human microbiome samples by body site using all-vs-all k-mer distance matrices from Mash, Libra, and Simka, then compares how each tool's metric (MinHash distance, Libra's log-weighted cosine similarity, Simka's Jaccard-ab and Bray-Curtis) resolves body-site structure.

## When to use

Use when you have paired-end WGS or simulated long-read metagenome samples with known body-site labels and need to compare how k-mer-based tools cluster them by origin.

## When NOT to use

- Your goal is single-pair sample comparison rather than all-vs-all clustering across many samples — Libra/Mash/Simka matrices are designed for all-vs-all comparison
- Samples lack known body-site or origin labels, so clustering outcomes cannot be validated against expected groupings
- You need strain- or variant-level resolution rather than metagenome-level community composition/abundance comparison

## Inputs

- 48 paired HMP whole-genome shotgun (WGS) read files (FASTA/FASTQ) with body-site metadata
- 49-sample simulated CAMI 'toy' Pacific Biosciences (PacBio) read dataset of HMP-derived samples
- Body-site labels (e.g., skin, stool, fornix) for sample coloring/clustering validation

## Outputs

- Mash all-vs-all MinHash distance matrix
- Libra log-weighted cosine similarity score matrix (written to score_dir)
- Simka Jaccard-ab distance matrix
- Simka Bray-Curtis distance matrix
- Per-tool body-site cluster assignments and a cross-tool clustering comparison (e.g., Fig. 4 / Fig. S2 style)

## How to apply

Obtain the 48 HMP paired whole-genome shotgun read files (or the 49-sample CAMI toy PacBio set) and run three tools on the same reads. For Mash, sketch each sample by unique k-mers and compute the all-vs-all MinHash distance matrix; for Libra, preprocess FASTA/FASTQ files (k-mer size 20 by default), then run scoring in MAP mode (m=MAP) with the default COSINESIMILARITY algorithm and LOGARITHM weighting, writing the score matrix with -o; for Simka, compute both Jaccard-ab and Bray-Curtis distance matrices. This yields four distance matrices (Mash, Libra, Simka Jaccard-ab, Simka Bray-Curtis) for the HMP WGS comparison, or three for the CAMI toy set. Hierarchically cluster the samples for each matrix and color/label them by known body-site origin. Rationale: Libra's cosine similarity uses all reads' k-mer abundance normalized for sequencing depth, whereas Mash's sketching and Simka's presence/absence or normalized metrics may lose resolution. Validate against the paper's reported pattern: on HMP WGS reads, Mash and Libra enhance body-site clustering (Mash with decreased resolution vs Libra), while Simka misclusters some skin and stool samples with fornix samples; on the CAMI PacBio toy data, all tools cluster broadly by body site with small shared misclassifications attributed to PacBio error rate.

## Related tools

- **Libra** (Computes the all-vs-all k-mer abundance cosine-similarity score matrix in MAP mode with COSINESIMILARITY scoring and LOGARITHM weighting; runs on a Hadoop framework) — https://github.com/iychoi/libra
- **Mash** (Sketches each sample by unique k-mers and emits the pairwise MinHash distance matrix for comparison with Libra)
- **Simka** (Computes Jaccard-ab and Bray-Curtis distance matrices over the same read set as comparator metrics)
- **Hadoop** (Scalable MapReduce framework underlying Libra's preprocessing and scoring (hadoop jar libra-all.jar))

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -o /index_dir /source_dir && hadoop jar libra-all.jar core -s COSINESIMILARITY -m MAP -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- On HMP WGS reads, Libra's matrix clusters samples by body site with higher resolution than Mash's MinHash matrix (Mash shows decreased resolution)
- On HMP WGS reads, both Simka Jaccard-ab and Bray-Curtis matrices miscluster some skin and stool samples with fornix samples
- On the CAMI toy PacBio dataset, every tool's clustering places samples broadly by body site, with only small misclassifications shared across all tools
- Libra score matrix is produced in score_dir with default COSINESIMILARITY algorithm and LOGARITHM weighting in MAP mode
- Each distance matrix is square, all-vs-all over the full sample set (48 HMP or 49 CAMI samples) with self-distance zero on the diagonal

## Limitations

- Presence/absence distance metrics and read count normalization (as in some comparator tools) greatly diminish the resolution of large-scale comparative analyses
- Increased PacBio error rate in the CAMI toy dataset causes small, shared misclassifications across all tools
- Simka's Jaccard-ab and Bray-Curtis metrics fail to cluster HMP WGS samples by body site
- Libra group/indexing defaults (e.g., 10GB group size, 20 max groups) may need adjustment for very large input sets

## Evidence

- [results] mash/libra clustering finding: [evidence span withheld — anchor status: not_found]
- [results] simka failure: [evidence span withheld — anchor status: not_found]
- [abstract] libra cosine similarity: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [methods] scoring options: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] scoring run: [evidence span withheld — anchor status: external_doc]
- [abstract] resolution caveat: "such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
