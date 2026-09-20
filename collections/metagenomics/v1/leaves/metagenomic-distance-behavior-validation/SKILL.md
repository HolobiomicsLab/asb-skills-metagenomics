---
name: metagenomic-distance-behavior-validation
description: Use when you need to reproduce, reimplement, or validate scalable all-vs-all
  metagenome distance computation from k-mer frequency vectors, or sanity-check that
  a distance implementation yields near-zero self-distance.
when_to_use_negative:
- When the goal is pairwise comparison of only a few sample pairs — the sweep-line
  design is built for massive all-vs-all comparison and per-pair tools may be simpler
- When k-mer frequencies are already aggregated into a feature table and the distance
  matrix can be computed directly in memory (e.g. small samples with scipy/sklearn
  cosine distance)
- When presence/absence or sketch-based comparison (e.g. Mash/Sourmash MinHash) is
  acceptable — the paper notes such data reduction methods 'greatly diminish the resolution'
  but they are far cheaper
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Implements the COSINESIMILARITY all-vs-all scoring over the k-mer inverted
    index, emitting the distance matrix
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework that executes the sweep line (MAP mode, or REDUCE
    for large indexes) across k-mer postings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:050ba7314907e1129d0c5458a7d1b34f8193d4c51cb67b2e4be586ba588ae643
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metagenomic-distance-behavior-validation

## Summary

Reconstruct and validate Libra's all-vs-all pairwise distance computation, in which a Hadoop MapReduce sweep line over a k-mer inverted index accumulates dot-product (D) and per-sample magnitude (M) terms for every sample pair in a single pass, yielding a cosine similarity distance matrix.

## When to use

Use when you need to reproduce, reimplement, or validate scalable all-vs-all metagenome distance computation from k-mer frequency vectors, or sanity-check that a distance implementation yields near-zero self-distance.

## When NOT to use

- When the goal is pairwise comparison of only a few sample pairs — the sweep-line design is built for massive all-vs-all comparison and per-pair tools may be simpler
- When k-mer frequencies are already aggregated into a feature table and the distance matrix can be computed directly in memory (e.g. small samples with scipy/sklearn cosine distance)
- When presence/absence or sketch-based comparison (e.g. Mash/Sourmash MinHash) is acceptable — the paper notes such data reduction methods 'greatly diminish the resolution' but they are far cheaper

## Inputs

- Inverted index of k-mer frequency vectors per sample (directory /index_dir, built by Libra preprocess from FASTA/FASTQ files)
- Index options such as k-mer size (-k) and kmer filter choice (-f NONE | STDDEV | STDDEV2 | NOTUNIQUE)

## Outputs

- All-vs-all cosine distance matrix between metagenome samples written to /score_dir
- Per-pair accumulated dot product (D) and magnitude (M) terms from the MapReduce sweep

## How to apply

First build (or load from Libra's upstream HM indexing step) an inverted index keyed by k-mer that stores each sample's k-mer frequency vector, produced by the preprocess command into /index_dir. Then run Libra's scoring command ('HM -o /score_dir /index_dir') with the COSINESIMILARITY scoring algorithm (s=COSINESIMILARITY, the default) to launch the all-vs-all comparison. Instead of scanning every vector pair separately, execute the pairwise scoring in Hadoop MAP mode (m=MAP, the default), which sweeps k-mers once and accumulates the dot product D and per-sample magnitudes M for every sample pair in linear time, falling back to REDUCE mode for large indexes. Optionally select a weighting scheme with -w: LOGARITHM (1+log f, the default), NATURAL (raw k-mer frequency f), or BOOLEAN presence/absence. The rationale is that Cosine Similarity compares metagenomes using sequence composition and abundance while normalizing for sequencing depth, and the sweep-line accumulation makes the n(n-1)/2 pairwise comparison feasible for massive datasets such as the 43 Tara Ocean Viromes. Finally, validate on small test datasets where a dataset compared against a duplicate of itself must yield a distance below 0.0001.

## Related tools

- **Libra** (Implements the COSINESIMILARITY all-vs-all scoring over the k-mer inverted index, emitting the distance matrix) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework that executes the sweep line (MAP mode, or REDUCE for large indexes) across k-mer postings)

## Examples

```
hadoop jar libra-all.jar core -s COSINESIMILARITY -m MAP -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- A dataset compared against a duplicate of itself yields a distance below 0.0001
- Implementation is validated on small test datasets before scaling up
- Distance matrix has dimensions n x n for n samples with symmetric off-diagonal entries and zero diagonal for identical inputs
- Single scan of all k-mer vectors suffices: the MAP-mode sweep touches each k-mer posting once while populating D and M for all pairs (linear-time behavior rather than per-pair scans)
- Using -w NATURAL vs -w LOGARITHM vs -w BOOLEAN changes the matrix, confirming the weighting scheme is actually applied

## Limitations

- MAP run mode may need to fall back to REDUCE mode for large indexes
- Methods such as data reduction, read count normalization, and presence/absence distance metrics diminish the resolution of large-scale comparative analyses, so metric choice matters
- Indexing depends on upstream k-mer choices (k-mer size, kmer filter algorithm NONE | STDDEV | STDDEV2 | NOTUNIQUE) which affect the distance matrix

## Evidence

- [other] Libra computes all pairwise sample distances with a sweep line algorithm that requires only a single scan of all k-mer vectors: [evidence span withheld — anchor status: not_found]
- [abstract] Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [methods] s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] m : run mode. MAP (default) | REDUCE: "- m : run mode. MAP (default) | REDUCE
- w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL
- o : output directory"
- [abstract] Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
