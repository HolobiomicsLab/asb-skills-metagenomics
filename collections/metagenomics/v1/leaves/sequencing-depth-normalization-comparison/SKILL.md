---
name: sequencing-depth-normalization-comparison
description: Use when pairwise metagenome distances must resolve communities differing
  in sequencing depth, dominant-organism abundance, or genetic composition, and MinHash
  sketches (Mash) or presence/absence metrics (Simka Jaccard) compress or saturate
  those differences.
when_to_use_negative:
- Input is already a k-mer feature table or abundance matrix rather than raw read
  files (Libra indexes reads directly)
- The analysis requires presence/absence or MinHash-sketch semantics only, and abundance-resolution
  of depth/abundance gradients is not the goal (Mash or Simka presence/absence metrics
  suffice)
- Datasets are too large for a single machine and no Hadoop/CyVerse infrastructure
  is available, since Libra relies on a Hadoop MapReduce architecture
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Builds the k-mer count index and computes Cosine Similarity all-vs-all distance
    matrices with selectable NATURAL/LOGARITHM/BOOLEAN weighting
  repo: https://github.com/iychoi/libra
- name: Mash
  role: Baseline MinHash distance metric from unique k-mer sketches used to contrast
    Libra's abundance-aware distances
- name: Simka
  role: Baseline pairwise distance matrices using Jaccard-ab and Bray-Curtis metrics
    for comparison of resolution behavior
- name: Hadoop
  role: Scalable MapReduce framework underpinning Libra's k-mer counting and scoring
    runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:a6560c3fb41b9203eabfda9f7f727a58d8a5693ad08bf5f5bc141c2fb01a4139
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# sequencing-depth-normalization-comparison

## Summary

Compare metagenomic samples with Libra's Cosine Similarity k-mer abundance distance, choosing NATURAL (raw-abundance) or LOGARITHM weighting to control how sequencing depth and dominant-organism abundance influence all-vs-all pairwise distances, as validated on simulated 454 mock-community series (mock 1–4).

## When to use

Use when pairwise metagenome distances must resolve communities differing in sequencing depth, dominant-organism abundance, or genetic composition, and MinHash sketches (Mash) or presence/absence metrics (Simka Jaccard) compress or saturate those differences.

## When NOT to use

- Input is already a k-mer feature table or abundance matrix rather than raw read files (Libra indexes reads directly)
- The analysis requires presence/absence or MinHash-sketch semantics only, and abundance-resolution of depth/abundance gradients is not the goal (Mash or Simka presence/absence metrics suffice)
- Datasets are too large for a single machine and no Hadoop/CyVerse infrastructure is available, since Libra relies on a Hadoop MapReduce architecture

## Inputs

- FASTA/FASTQ metagenomic read files (e.g. simulated 454 reads from the staggered HM-277D mock community, mock 1–4)
- Libra k-mer count index directory produced by the preprocess step

## Outputs

- All-vs-all pairwise distance matrix from Libra scoring with NATURAL weighting
- All-vs-all pairwise distance matrix from Libra scoring with LOGARITHM weighting
- Optionally comparable Mash MinHash distance matrix and Simka Jaccard-ab / Bray-Curtis matrices

## How to apply

First build the Libra k-mer count index from FASTA/FASTQ reads with the preprocess step (e.g. k-mer size 20, NOTUNIQUE default filter). Then run Libra scoring twice on the same index: once with the default COSINESIMILARITY algorithm and NATURAL weighting (-w NATURAL) to obtain distances in raw-abundance space, and once with LOGARITHM weighting (-w LOGARITHM), which dampens dominance by highly abundant organisms. On the simulated 454 mock-community composition series (mock 1–4), logarithmic weighting resolves communities with a stepwise increase in pairwise distance, whereas Mash distances compress and Simka Bray-Curtis/Jaccard hit maximum difference between mocks 3 and 4. Interpret the NATURAL matrix when absolute abundance differences matter (e.g. read-duplication or depth-series responses) and the LOGARITHM matrix when composition gradient resolution is the goal. Confirm known qualitative patterns such as the stepwise mock1–mock4 distance increase and flat Simka distances across the 0.5/1/5/10 M depth series.

## Related tools

- **Libra** (Builds the k-mer count index and computes Cosine Similarity all-vs-all distance matrices with selectable NATURAL/LOGARITHM/BOOLEAN weighting) — https://github.com/iychoi/libra
- **Mash** (Baseline MinHash distance metric from unique k-mer sketches used to contrast Libra's abundance-aware distances)
- **Simka** (Baseline pairwise distance matrices using Jaccard-ab and Bray-Curtis metrics for comparison of resolution behavior)
- **Hadoop** (Scalable MapReduce framework underpinning Libra's k-mer counting and scoring runs)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Mock 1–4 pairwise distances under LOGARITHM weighting show a stepwise increase in pairwise distance across the composition series
- NATURAL-weighted distances respond to read duplication (measurable change), while LOGARITHM and other metrics show no measurable change under duplication
- Re-running scoring with -w NATURAL vs -w LOGARITHM on the same index yields two distinct matrices rather than identical outputs
- Simka Jaccard-ab/Bray-Curtis distances remain flat across the 0.5/1/5/10 M sequencing-depth series, contrasting with depth-normalized Libra Cosine Similarity distances
- Scoring output is an all-vs-all matrix covering every sample pair with COSINESIMILARITY as the reported scoring algorithm

## Limitations

- Distances compress or saturate with non-Libra metrics: Mash distances compress across mock 1–4 and Simka Bray-Curtis/Jaccard reach maximum difference between mocks 3 and 4, so cross-metric agreement should not be expected
- Read-count normalization and presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses, so BOOLEAN weighting or sketch-based metrics lose abundance information
- The k-mer filter and grouping options (e.g. NOTUNIQUE default filter, 10GB default group size, 20 max groups) affect index structure and must be fixed for comparable scoring runs

## Evidence

- [other] On the simulated 454 mock-community composition series (mock 1–4), Libra with logarithmic weighting resolves the communities with a stepwise increase in pairwise distance: [evidence span withheld — anchor status: not_found]
- [readme] README scoring command with weighting option: [evidence span withheld — anchor status: external_doc]
- [readme] README weighting and scoring algorithm options: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [abstract] Abstract on distance calculation and depth normalization: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [abstract] Abstract finding on resolution loss from normalization metrics: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
