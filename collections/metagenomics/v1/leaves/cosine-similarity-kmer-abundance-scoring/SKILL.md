---
name: cosine-similarity-kmer-abundance-scoring
description: Use when you need all-vs-all distance matrices between FASTA/FASTQ metagenomic
  samples that retain both genetic-distance and abundance signal, e.g. for clustering
  16S rRNA or WGS datasets at high resolution.
when_to_use_negative:
- Presence/absence or sketch-based comparison is acceptable and full-resolution abundance
  signal is not needed (e.g. rapid Mash/Sourmash screening)
- You need abundant-metric comparisons other than cosine similarity — Libra also offers
  BRAYCURTIS and JENSENSHANNON scoring algorithms, so cosine is a choice, not a requirement
- Input is already a k-mer count or feature table rather than raw FASTA/FASTQ reads,
  since Libra computes k-mer abundance directly from reads
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Libra
  role: Performs the all-vs-all cosine-similarity scoring on k-mer abundance from
    raw reads via Hadoop MapReduce
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which Libra's preprocess and scoring steps
    run
- name: Mash
  role: Baseline MinHash method used for comparison; indexes samples by unique k-mers
    and shows lower clustering resolution on HMP 16S rRNA data
- name: Simka
  role: Baseline tool computing Jaccard-abundance and Bray-Curtis distance matrices
    used to contextualize Libra cosine clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:b976418eb7d26a8f509c16d5f49f4cbdd2565c70c5046011fe768771ff69a3fd
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cosine-similarity-kmer-abundance-scoring

## Summary

Score all-vs-all metagenome similarity with Libra using Cosine Similarity over full k-mer abundance profiles, normalized for sequencing depth via logarithmic weighting, to preserve clustering resolution that data-reduction and presence/absence metrics lose.

## When to use

Use when you need all-vs-all distance matrices between FASTA/FASTQ metagenomic samples that retain both genetic-distance and abundance signal, e.g. for clustering 16S rRNA or WGS datasets at high resolution.

## When NOT to use

- Presence/absence or sketch-based comparison is acceptable and full-resolution abundance signal is not needed (e.g. rapid Mash/Sourmash screening)
- You need abundant-metric comparisons other than cosine similarity — Libra also offers BRAYCURTIS and JENSENSHANNON scoring algorithms, so cosine is a choice, not a requirement
- Input is already a k-mer count or feature table rather than raw FASTA/FASTQ reads, since Libra computes k-mer abundance directly from reads

## Inputs

- FASTA/FASTQ metagenomic sample files (e.g. 48 matched HMP 16S rRNA amplicon samples)
- Libra k-mer index directory produced by the preprocess step

## Outputs

- All-vs-all cosine similarity score matrix between samples, written to the -o score directory
- Sample clusters derived from the distance matrix (e.g. HMP 16S rRNA clusters comparable to Fig. 3B)

## How to apply

Preprocess FASTA/FASTQ inputs with the Libra preprocess step to build k-mer indices (e.g. -k 20; k-mer filter NOTUNIQUE by default), then run scoring with 'libra core' on the index directory. Keep the defaults s=COSINESIMILARITY, w=LOGARITHM, m=MAP, writing the all-vs-all score matrix to the -o output directory; Libra runs on Hadoop MapReduce and uses all reads rather than sketching or subsampling. Rationale: the article shows that data reduction, read count normalization, and presence/absence metrics greatly diminish clustering resolution, whereas Cosine Similarity on sequence composition and abundance, normalized for sequencing depth, distinguishes metagenomes by both community composition and abundance. Validate by clustering the resulting matrix (e.g. the 48 matched HMP 16S rRNA samples) and confirming clustering by broad sample category at higher resolution than Mash (Fig. 3A vs 3B), while both Simka Jaccard-abundance and Bray-Curtis metrics fail to cluster by broad categories.

## Related tools

- **Libra** (Performs the all-vs-all cosine-similarity scoring on k-mer abundance from raw reads via Hadoop MapReduce) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which Libra's preprocess and scoring steps run)
- **Mash** (Baseline MinHash method used for comparison; indexes samples by unique k-mers and shows lower clustering resolution on HMP 16S rRNA data)
- **Simka** (Baseline tool computing Jaccard-abundance and Bray-Curtis distance matrices used to contextualize Libra cosine clustering)

## Examples

```
hadoop jar libra-all.jar core -s COSINESIMILARITY -w LOGARITHM -m MAP -o /score_dir /index_dir
```

## Evaluation signals

- Score matrix is all-vs-all: one similarity value for every sample pair (N×N for N input samples)
- Scoring run used s=COSINESIMILARITY with w=LOGARITHM weighting and m=MAP run mode (Libra defaults)
- Clustering of the 48 HMP 16S rRNA samples reproduces broad-category clustering at higher resolution than Mash (Fig. 3A vs Fig. 3B)
- Simka Jaccard-abundance and Bray-Curtis matrices fail to cluster HMP samples by broad categories, while the Libra cosine matrix does — a negative control for the resolution claim
- Preprocess step completed with a defined k-mer size (e.g. -k 20) and NOTUNIQUE k-mer filter before scoring

## Limitations

- Data-reduction, read count normalization, and presence/absence distance metrics (as used in sketching tools like Mash) greatly diminish clustering resolution; this skill's benefit depends on using all reads
- Requires a Hadoop environment; scale-out via MapReduce is the design point, so single-machine runs may be cumbersome
- Choice of k-mer size and k-mer filter (NONE | STDDEV | STDDEV2 | NOTUNIQUE) during preprocess affects the abundance profile and thus scores

## Evidence

- [abstract] Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [abstract] Data reduction and presence/absence metrics diminish resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [readme] Scoring option defaults: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] Scoring command and weighting default: [evidence span withheld — anchor status: external_doc]
- [intro] HMP validation dataset: "we show Libra's ability to distinguish metagenomes by both community composition and abundance using 48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project"
