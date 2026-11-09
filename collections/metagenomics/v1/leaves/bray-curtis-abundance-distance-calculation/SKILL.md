---
name: bray-curtis-abundance-distance-calculation
description: Use when you need an abundance-based all-vs-all distance matrix over
  sequencing samples (reads or assembled contigs) to benchmark abundance-ignoring
  methods like Mash or normalization-limited methods like Simka.
when_to_use_negative:
- Input is not raw reads or contigs but an already-computed feature/abundance table
  where Bray-Curtis can be computed directly without Libra's k-mer indexing
- The analysis goal is purely composition-based comparison where abundance should
  be ignored (Mash sketching is the article's comparator)
- The metric requires normalization to minimum sample depth in a way that collapses
  low-count contig datasets — the article shows Simka's normalization reduced all
  assembled-contig samples to the smallest contig count, yielding near-maximum distances
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Runs the BRAYCURTIS scoring algorithm over precomputed k-mer indexes (Hadoop
    MapReduce) to emit the abundance-based pairwise distance matrix
  repo: iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which Libra's preprocess and scoring stages
    execute
- name: Simka
  role: Alternative tool that also computes Bray-Curtis distance matrices between
    metagenomes, used to cross-check Libra's Bray-Curtis results on HMP samples
- name: Mash
  role: Abundance-ignoring MinHash baseline whose matrices are compared against the
    Bray-Curtis matrix to assess clustering resolution
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:2bd6db5de78b37e0098157e286f3f1e30304f8fe9c528cc2a805f824326daeb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bray-curtis-abundance-distance-calculation

## Summary

Computes abundance-aware Bray-Curtis distance matrices between metagenomic samples from k-mer abundance profiles, using Libra's BRAYCURTIS scoring algorithm as a comparison point to its default Cosine Similarity metric.

## When to use

Use when you need an abundance-based all-vs-all distance matrix over sequencing samples (reads or assembled contigs) to benchmark abundance-ignoring methods like Mash or normalization-limited methods like Simka.

## When NOT to use

- Input is not raw reads or contigs but an already-computed feature/abundance table where Bray-Curtis can be computed directly without Libra's k-mer indexing
- The analysis goal is purely composition-based comparison where abundance should be ignored (Mash sketching is the article's comparator)
- The metric requires normalization to minimum sample depth in a way that collapses low-count contig datasets — the article shows Simka's normalization reduced all assembled-contig samples to the smallest contig count, yielding near-maximum distances

## Inputs

- FASTA/FASTQ metagenomic sample files (e.g., 48-sample HMP 16S rRNA or HMP WGS collections, or assembled contig sets)
- Libra k-mer index directory produced by the preprocess step (-o /index_dir)

## Outputs

- All-vs-all pairwise Bray-Curtis distance matrix (e.g., hmp_wgs_distance_matrices.tsv with one value per sample pair)
- Clustered sample dendrograms/heatmaps from the Bray-Curtis matrix

## How to apply

First build k-mer indexes for the FASTA/FASTQ samples with Libra's HM preprocessor (e.g., hadoop jar libra-all.jar preprocess -k 20 -o /index_dir /source_dir). Then run Libra scoring with the s=BRAYCURTIS scoring algorithm option (instead of the default COSINESIMILARITY), keeping the default LOGARITHM weighting and choosing run mode m=MAP (default) or m=REDUCE, writing the distance matrix to the directory given by -o. Bray-Curtis incorporates k-mer abundance, so unlike Mash's presence/absence-style MinHash sketching, it retains abundance information; the article's motivation is that abundance-preserving metrics give better clustering resolution. On HMP WGS assembled contigs, expect Bray-Curtis to still be constrained by count normalization effects across samples. Judge success by producing a complete all-vs-all pairwise distance matrix and clustering samples to check that they group by known categories (e.g., HMP body site).

## Related tools

- **Libra** (Runs the BRAYCURTIS scoring algorithm over precomputed k-mer indexes (Hadoop MapReduce) to emit the abundance-based pairwise distance matrix) — iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which Libra's preprocess and scoring stages execute)
- **Simka** (Alternative tool that also computes Bray-Curtis distance matrices between metagenomes, used to cross-check Libra's Bray-Curtis results on HMP samples)
- **Mash** (Abundance-ignoring MinHash baseline whose matrices are compared against the Bray-Curtis matrix to assess clustering resolution)

## Examples

```
hadoop jar libra-all.jar core -s BRAYCURTIS -m REDUCE -o /score_dir /index_dir
```

## Evaluation signals

- The output directory given by -o contains a complete distance matrix with one pairwise value for every sample pair (e.g., a 48x48 matrix for the HMP collections)
- Re-running with the default s=COSINESIMILARITY vs s=BRAYCURTIS yields different matrices, confirming the abundance-based option was actually applied
- Clustering the Bray-Curtis matrix groups samples by known broad categories (e.g., HMP body sites), with better resolution than Mash's matrix
- On assembled-contig inputs (Fig. S3 setting), the matrix should be checked for distance compression rather than blindly treated as high-resolution, since contig-count normalization can inflate all pairwise distances
- The scoring log confirms the run used -s BRAYCURTIS with expected defaults (w=LOGARITHM, m=MAP or REDUCE)

## Limitations

- Abundance/normalization-based distance metrics greatly diminish resolution on some inputs — Simka's Bray-Curtis failed to distinguish assembled HMP WGS datasets, scoring all sample-to-sample distances at the maximum
- Requires a Hadoop environment and a prior preprocess (k-mer indexing) step; k-mer size and k-mer filter (NONE | STDDEV | STDDEV2 | NOTUNIQUE) choices affect the index
- Bray-Curtis retains abundance but the article argues Cosine Similarity is the better distance metric for comparing metagenomes considering genetic distance and microbial abundance simultaneously

## Evidence

- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [abstract] Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [abstract] methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [readme] hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir: [evidence span withheld — anchor status: external_doc]
