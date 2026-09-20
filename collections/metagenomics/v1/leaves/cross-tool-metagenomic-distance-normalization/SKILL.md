---
name: cross-tool-metagenomic-distance-normalization
description: Use when comparing multiple metagenome distance tools (Libra, Mash, Simka)
  on the same sample set to assess how data reduction, read-count normalization, or
  presence/absence metrics distort clustering by a known metadata factor such as body
  site.
when_to_use_negative:
- Input is assembled contigs when the goal is to demonstrate Libra's abundance-aware
  clustering — Libra requires reads to calculate k-mer abundance and shows reduced
  resolution on contigs
- Samples have wildly unequal sizes and the chosen tool normalizes counts to the minimum
  sample count (Simka behavior), which collapses all pairwise distances to the maximum
- The analysis goal is presence/absence comparison only and abundance-driven separation
  is the expected signal
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Libra
  role: Builds Hadoop HM k-mer indexes and computes COSINESIMILARITY (default) and
    BRAYCURTIS distance matrices from all reads
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework underlying Libra's preprocessing and scoring
    jobs
- name: Mash
  role: Sketches samples and computes all-vs-all MinHash distances used as the composition-only
    baseline that still clusters body sites
- name: Simka
  role: Computes Jaccard-ab and Bray-Curtis distance matrices between metagenomes,
    normalizing counts to the minimum contig count
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:71c1ebb7bcbef1c5918682dec9efc014d470235356a693af47c9844f5f69f3ed
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-tool-metagenomic-distance-normalization

## Summary

Benchmark how distance metrics and count-normalization choices (Libra Cosine Similarity/Bray-Curtis on all reads, Mash MinHash sketches, Simka Jaccard-ab/Bray-Curtis with minimum-count normalization) affect all-vs-all metagenome clustering, e.g. demonstrating that assembled contigs strip abundance and collapse Simka distances.

## When to use

Use when comparing multiple metagenome distance tools (Libra, Mash, Simka) on the same sample set to assess how data reduction, read-count normalization, or presence/absence metrics distort clustering by a known metadata factor such as body site.

## When NOT to use

- Input is assembled contigs when the goal is to demonstrate Libra's abundance-aware clustering — Libra requires reads to calculate k-mer abundance and shows reduced resolution on contigs
- Samples have wildly unequal sizes and the chosen tool normalizes counts to the minimum sample count (Simka behavior), which collapses all pairwise distances to the maximum
- The analysis goal is presence/absence comparison only and abundance-driven separation is the expected signal

## Inputs

- Metagenomic sample sets in FASTA/FASTQ (e.g. 48-sample Human Microbiome Project WGS reads or assembled contigs)
- Sample metadata (body site or other grouping variable) for cluster validation
- Libra HM k-mer indexes built from the sample directory

## Outputs

- Libra COSINESIMILARITY distance matrix (from -o /score_dir)
- Libra BRAYCURTIS distance matrix
- Mash all-vs-all MinHash distance matrix
- Simka distance matrix (Jaccard-ab and Bray-Curtis) with minimum-count normalization
- hmp_wgs_distance_matrices.tsv with one pairwise distance value for every sample pair

## How to apply

Load the same sample collection (e.g. the 48-sample HMP WGS assembled contig sets) into all tools. Build k-mer indexes with Libra's HM preprocessor, then run Libra scoring in REDUCE mode twice: once with the default COSINESIMILARITY scoring, LOGARITHM weighting, and -o /score_dir, and once with the BRAYCURTIS scoring algorithm option, producing two distance matrices. Sketch all samples with Mash for all-vs-all MinHash distances, and run Simka on the same contig sets to get its Jaccard-ab and Bray-Curtis matrices, noting Simka normalizes counts to the minimum contig count across samples. Cluster samples from each matrix and compare against known groupings (distinct body sites) to reveal where normalization or presence/absence metrics reduce resolution. In the reference result, Mash separates body sites while Simka scores all sample-to-sample distances at the maximum and Libra shows discrepancies on contigs, because Libra's abundance signal requires reads rather than assembled contigs.

## Related tools

- **Libra** (Builds Hadoop HM k-mer indexes and computes COSINESIMILARITY (default) and BRAYCURTIS distance matrices from all reads) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework underlying Libra's preprocessing and scoring jobs)
- **Mash** (Sketches samples and computes all-vs-all MinHash distances used as the composition-only baseline that still clusters body sites)
- **Simka** (Computes Jaccard-ab and Bray-Curtis distance matrices between metagenomes, normalizing counts to the minimum contig count)

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir && hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- The three matrices reproduce the reported qualitative pattern: Mash clusters samples by distinct body site, Libra shows discrepancies with reduced resolution, Simka assigns near-maximum sample-to-sample distances to all pairs
- hmp_wgs_distance_matrices.tsv exists and contains one pairwise distance value for every sample pair (complete 48x48 symmetric matrix)
- Libra REDUCE-mode scoring with default COSINESIMILARITY and LOGARITHM weighting completes and emits a distance matrix to /score_dir
- Simka distances on assembled contigs sit at the maximum value, consistent with minimum-contig-count normalization reducing every sample

## Limitations

- Libra's abundance-aware distances degrade on assembled contigs because Libra uses all reads to calculate k-mer abundance
- Simka's normalization to the minimum contig count across samples can drive all sample-to-sample distances to the maximum, destroying discrimination
- Data reduction, read count normalization, and presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses

## Evidence

- [results] Simka could not distinguish assembled datasets: [evidence span withheld — anchor status: not_found]
- [abstract] Libra uses all reads for k-mer abundance: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset"
- [abstract] Normalization diminishes resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution"
- [methods] Libra scoring options: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] README scoring invocation: [evidence span withheld — anchor status: external_doc]
