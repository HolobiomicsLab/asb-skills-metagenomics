---
name: hmp-body-site-clustering-evaluation
description: Use when evaluating how k-mer distance metrics (Cosine Similarity, Bray-Curtis,
  MinHash, Simka) cluster metagenomic samples by body site, especially to test whether
  analysis of assembled contigs instead of reads degrades separation.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled contigs — the known failure
  mode being reproduced (Simka near-maximum distances, Libra low resolution) applies
  to the contig-based setting
- Goal is abundance-sensitive comparison where reads and their k-mer counts must be
  fully used (Libra's intended mode, which requires reads rather than assemblies)
- Dataset is not a body-site-annotated collection such as HMP, so body-site clustering
  cannot serve as an acceptance check
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Builds k-mer indexes (HM preprocessor, k=20) and computes COSINESIMILARITY
    and BRAYCURTIS all-vs-all distance matrices in REDUCE mode with LOGARITHM weighting
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which Libra preprocessing and scoring are
    executed (hadoop jar libra-all.jar ...)
- name: Mash
  role: Sketches the contigs and computes all-vs-all MinHash distances that cluster
    samples by distinct body site (the positive control)
- name: Simka
  role: Computes the distance matrix between metagenomes on the same contig sets,
    normalizing counts to the minimum contig count (the degraded-resolution case)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:ff46656c542a7ec40d153a3246c55617e3b4942c50c6d323f3f62f67a7c9b443
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hmp-body-site-clustering-evaluation

## Summary

Reproduces the Libra paper's assembled-contigs comparison on the 48-sample Human Microbiome Project (HMP) WGS dataset, benchmarking Mash, Libra (Cosine Similarity and Bray-Curtis), and Simka distance matrices for clustering samples by body site. It demonstrates that data reduction (contigs instead of reads) and count normalization strip clustering resolution.

## When to use

Use when evaluating how k-mer distance metrics (Cosine Similarity, Bray-Curtis, MinHash, Simka) cluster metagenomic samples by body site, especially to test whether analysis of assembled contigs instead of reads degrades separation.

## When NOT to use

- Input is raw sequencing reads rather than assembled contigs — the known failure mode being reproduced (Simka near-maximum distances, Libra low resolution) applies to the contig-based setting
- Goal is abundance-sensitive comparison where reads and their k-mer counts must be fully used (Libra's intended mode, which requires reads rather than assemblies)
- Dataset is not a body-site-annotated collection such as HMP, so body-site clustering cannot serve as an acceptance check

## Inputs

- HMP WGS assembled-contig datasets (48-sample Human Microbiome Project collection, FASTA contig files per sample)
- Libra k-mer index directory produced by the preprocess step (-k 20)
- Mash sketches of the same contig sets
- Simka count-normalized contig inputs

## Outputs

- Libra Cosine Similarity distance matrix (-o /score_dir)
- Libra Bray-Curtis distance matrix
- Mash all-vs-all MinHash distance matrix
- Simka distance matrix between metagenomes
- hmp_wgs_distance_matrices.tsv with one pairwise distance value for every sample pair
- Body-site clustering of the 48 samples from each matrix

## How to apply

Load the HMP WGS assembled-contig datasets (48-sample Human Microbiome Project collection) and build k-mer indexes with Libra's HM preprocessor (e.g. k=20), then run Libra scoring in REDUCE mode with the default COSINESIMILARITY algorithm, LOGARITHM weighting, and -o /score_dir; re-run with the BRAYCURTIS scoring option for the abundance-based matrix. Sketch the same contigs with Mash for all-vs-all MinHash distances, and run Simka to compute its distance matrix, normalizing counts to the minimum contig count across samples. Cluster samples from each matrix and check the qualitative pattern: Mash separates distinct body sites, Libra shows discrepancies with reduced resolution on contigs, and Simka assigns near-maximum sample-to-sample distances to all pairs because its normalization reduced every sample to the smallest contig count. The rationale is the paper's finding that data reduction, read count normalization, and presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses, whereas Libra uses all reads to calculate k-mer abundance.

## Related tools

- **Libra** (Builds k-mer indexes (HM preprocessor, k=20) and computes COSINESIMILARITY and BRAYCURTIS all-vs-all distance matrices in REDUCE mode with LOGARITHM weighting) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which Libra preprocessing and scoring are executed (hadoop jar libra-all.jar ...))
- **Mash** (Sketches the contigs and computes all-vs-all MinHash distances that cluster samples by distinct body site (the positive control))
- **Simka** (Computes the distance matrix between metagenomes on the same contig sets, normalizing counts to the minimum contig count (the degraded-resolution case))

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- hmp_wgs_distance_matrices.tsv exists and contains one pairwise distance value for every sample pair of the 48 HMP WGS samples
- Mash matrix clusters samples by distinct body site
- Libra Cosine Similarity matrix shows discrepancies with reduced resolution relative to the reads-based result
- Simka (Jaccard-ab and Bray-Curtis) matrix assigns near-maximum sample-to-sample distances to all pairs, reflecting minimum-count normalization
- Libra scoring completes with default COSINESIMILARITY / LOGARITHM in REDUCE mode and produces a second matrix when re-run with BRAYCURTIS

## Limitations

- On assembled contigs, Simka's normalization to the smallest contig count reduced every sample so it could not distinguish any of the assembled datasets
- Data reduction, read count normalization, and presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses
- Libra requires reads for full-resolution results; contig-based analysis yields reduced resolution

## Evidence

- [results] Simka normalization failure: [evidence span withheld — anchor status: not_found]
- [abstract] Resolution loss from reduction: "such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [intro] HMP dataset description: "48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project (HMP)"
- [readme] Scoring algorithm options: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [intro] Simka purpose: "Recently, Simka [15] was developed to compute a distance matrix between metagenomes"
