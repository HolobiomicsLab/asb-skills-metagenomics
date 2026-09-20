---
name: hadoop-mapreduce-metagenomic-scoring
description: Use when you need an all-vs-all distance/similarity matrix across many
  metagenomic FASTA/FASTQ read sets where data reduction or presence/absence metrics
  (e.g., Mash sketches, Simka Jaccard-ab) lose clustering resolution by community
  composition and abundance.
when_to_use_negative:
- You only need sketch-based proximity and compute is the bottleneck and resolution
  loss is acceptable — Mash is faster since it indexes samples by unique k-mers
- Input is a feature/abundance table rather than raw FASTA/FASTQ reads — Libra's preprocess
  and k-mer counting operate directly on read files
- A presence/absence or Jaccard-ab-based metric is explicitly desired (Simka-style
  comparison) rather than abundance-aware Cosine Similarity
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: 'Core tool: computes all-vs-all metagenome comparisons via Cosine Similarity
    on k-mer abundance using Hadoop MapReduce'
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework that Libra runs on for massive metagenome comparisons
- name: Mash
  role: 'Benchmark comparator: MinHash sketch-based all-vs-all distance matrix on
    the same read set'
- name: Simka
  role: 'Benchmark comparator: computes Jaccard-ab and Bray-Curtis distance matrices
    on the same read set'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:10dfbf533349ca6933756a6c9e006cf31ac4c5183f0e928ff51ee41df5792616
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hadoop-mapreduce-metagenomic-scoring

## Summary

Run Libra, a Hadoop MapReduce-based k-mer tool, to compute all-vs-all similarity/distance matrices between metagenomic samples using Cosine Similarity on k-mer abundance, enabling scalable clustering (e.g., of 48 HMP WGS read samples by body site).

## When to use

Use when you need an all-vs-all distance/similarity matrix across many metagenomic FASTA/FASTQ read sets where data reduction or presence/absence metrics (e.g., Mash sketches, Simka Jaccard-ab) lose clustering resolution by community composition and abundance.

## When NOT to use

- You only need sketch-based proximity and compute is the bottleneck and resolution loss is acceptable — Mash is faster since it indexes samples by unique k-mers
- Input is a feature/abundance table rather than raw FASTA/FASTQ reads — Libra's preprocess and k-mer counting operate directly on read files
- A presence/absence or Jaccard-ab-based metric is explicitly desired (Simka-style comparison) rather than abundance-aware Cosine Similarity

## Inputs

- Paired or single-end metagenomic FASTA/FASTQ read files (e.g., 48 HMP whole-genome shotgun read files)
- Hadoop/SBT installation capable of running libra-all.jar (Java-built jar, e.g., v1.2.8)
- Libra k-mer index directory produced by the preprocess step

## Outputs

- Libra score matrix (all-vs-all sample similarity matrix) written to the output score directory
- Sample clustering assignments (e.g., by body site) derived from the score matrix for comparison against other tools

## How to apply

First preprocess the raw FASTA/FASTQ reads by indexing k-mers (e.g., k=20) with `libra-all.jar preprocess`, optionally filtering non-unique k-mers (kmer filter NOTUNIQUE, default). Then run the scoring step in MAP mode (m=MAP) with the COSINESIMILARITY scoring algorithm (s, default) and LOGARITHM weighting (w, default), writing the score matrix to an output directory (-o score_dir). Cosine Similarity is chosen because it compares sequence composition and abundance while normalizing for sequencing depth, preserving resolution that presence/absence and sketch-based methods diminish. For benchmarking against Mash and Simka (e.g., HMP WGS body-site clustering), cluster the same samples from each tool's distance matrix and compare clustering assignments. Validate that Libra's matrix enhances body-site clustering where Mash shows decreased resolution and Simka (Jaccard-ab and Bray-Curtis) misclusters samples.

## Related tools

- **Libra** (Core tool: computes all-vs-all metagenome comparisons via Cosine Similarity on k-mer abundance using Hadoop MapReduce) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework that Libra runs on for massive metagenome comparisons)
- **Mash** (Benchmark comparator: MinHash sketch-based all-vs-all distance matrix on the same read set)
- **Simka** (Benchmark comparator: computes Jaccard-ab and Bray-Curtis distance matrices on the same read set)

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -o /index_dir /reads_dir; hadoop jar libra-all.jar core -m MAP -s COSINESIMILARITY -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- The scoring run completes in MAP mode with COSINESIMILARITY (default s) and LOGARITHM (default w) and emits a full 48x48 (N x N) symmetric score matrix in the -o score_dir
- Clustering samples by body site from the Libra matrix shows enhanced grouping compared with Mash (decreased resolution) and Simka, where skin and stool samples miscluster with fornix samples
- Preprocess produces index files grouped under the -s/-g settings (10GB group size, max 20 groups by default) without k-mer filter errors
- Re-running with the same k-mer size (e.g., k=20) and default options reproduces consistent scores (determinism check)
- Result matrices agree with the reported finding that data reduction and presence/absence metrics diminish comparative resolution relative to abundance-aware Cosine Similarity

## Limitations

- Requires a Hadoop environment; Libra is a Java jar executed with `hadoop jar`, adding infrastructure overhead
- Data reduction, read count normalization, and presence/absence distance metrics (as in competing methods) greatly diminish resolution of large-scale comparative analyses — Libra deliberately avoids them, so scores are not directly comparable to sketch-based distances
- K-mer filtering defaults to NOTUNIQUE; alternative filters (NONE, STDDEV, STDDEV2) change the index content and thus downstream scores
- Web-based use requires an iMicrobe/CyVerse account for data in the iMicrobe Data Commons

## Evidence

- [abstract] We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content.: "We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content."
- [abstract] Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [readme] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir: [evidence span withheld — anchor status: external_doc]
- [intro] 48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project (HMP): "48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project (HMP)"
