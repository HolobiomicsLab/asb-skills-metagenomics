---
name: silhouette-score-cluster-validation
description: Use when clustering tabular embeddings (e.g., 100-d Word2Vec vectors
  of FASTA files) with scikit-learn KMeans and you need a single numeric metric to
  verify the cluster assignment reproduces a reported reference score.
when_to_use_negative:
- Input is already a feature table or cluster label assignment — the score must be
  computed on the raw 100-d embeddings, not on aggregate statistics
- Data has not been preprocessed — MOVIS's data analysis core presupposes the data
  is quality controlled and filtered before clustering
- The clustering algorithm is OPTICS or another density method rather than the distance-based
  KMeans clustering MOVIS offers, and no comparable reference silhouette value exists
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides KMeans (3 centroids) for the optional data clustering step and silhouette_score
    for validation on the 100-d embeddings
  repo: https://github.com/AAnzel/MOVIS
- name: Gensim
  role: Trains the Word2Vec model that embeds each FASTA sequence into a 100-dimensional
    vector prior to clustering
  repo: https://github.com/AAnzel/MOVIS
- name: Biopython
  role: Parses each FASTA sequence from the archived BWWTP metaproteomics data before
    embedding
  repo: https://github.com/AAnzel/MOVIS
- name: MOVIS
  role: Modular time-series multi-omics tool whose sanitizing conventions and data
    clustering step (KMeans alongside OPTICS) frame this validation workflow
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:08b377d5766cc540ad0e6fbba79958ac7316b3ab51db865edd294351824f1aa2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Biopython
  - Gensim
  - MOVIS
  - Pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Silhouette score cluster validation

## Summary

Validates K-Means clustering quality on embedded time-series multi-omics vectors (e.g., 100-d Word2Vec FASTA embeddings from the BWWTP metaproteomics data in MOVIS) by computing scikit-learn's silhouette score and comparing it to a reference value (0.495 for k=3).

## When to use

Use when clustering tabular embeddings (e.g., 100-d Word2Vec vectors of FASTA files) with scikit-learn KMeans and you need a single numeric metric to verify the cluster assignment reproduces a reported reference score.

## When NOT to use

- Input is already a feature table or cluster label assignment — the score must be computed on the raw 100-d embeddings, not on aggregate statistics
- Data has not been preprocessed — MOVIS's data analysis core presupposes the data is quality controlled and filtered before clustering
- The clustering algorithm is OPTICS or another density method rather than the distance-based KMeans clustering MOVIS offers, and no comparable reference silhouette value exists

## Inputs

- Archived metaproteomics raw FASTA files from Data/cached/example_1/ (Herold et al. BWWTP data)
- 100-dimensional Word2Vec embeddings averaged per FASTA file (tabular dataset, 101 columns including temporal column)
- KMeans cluster labels (k=3) predicted on the embeddings

## Outputs

- Single silhouette score value (expected ≈ 0.495 for the reference task), written to silhouette_score.txt

## How to apply

First embed each archived FASTA file: parse sequences with Biopython, train a Gensim Word2Vec model to map sequences to 100-dimensional vectors, then average sequence vectors per file to yield a Pandas DataFrame with as many rows as FASTA files plus one temporal column (101 columns total). Fit scikit-learn KMeans with the chosen number of centroids (k=3 for the BWWTP Example 1 metaproteomics raw FASTA data), predict labels, and call sklearn.metrics.silhouette_score on the 100-d embeddings with those labels. Write the single resulting value to silhouette_score.txt. The rationale is that MOVIS offers distance-based clustering (KMeans) alongside OPTICS as the optional data clustering step, and the silhouette score quantifies cluster cohesion/separation on the same embedding space used for fitting, enabling direct comparison to the published reference of 0.495.

## Related tools

- **scikit-learn** (Provides KMeans (3 centroids) for the optional data clustering step and silhouette_score for validation on the 100-d embeddings) — https://github.com/AAnzel/MOVIS
- **Gensim** (Trains the Word2Vec model that embeds each FASTA sequence into a 100-dimensional vector prior to clustering) — https://github.com/AAnzel/MOVIS
- **Biopython** (Parses each FASTA sequence from the archived BWWTP metaproteomics data before embedding) — https://github.com/AAnzel/MOVIS
- **MOVIS** (Modular time-series multi-omics tool whose sanitizing conventions and data clustering step (KMeans alongside OPTICS) frame this validation workflow) — https://github.com/AAnzel/MOVIS

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import silhouette_score; labels = KMeans(n_clusters=3, random_state=42).fit_predict(X); score = silhouette_score(X, labels)
```

## Evaluation signals

- Computed silhouette score must match the reported reference value of 0.495 within a small numerical tolerance
- Output file silhouette_score.txt must exist and contain a numeric value close to 0.495
- Embedded dataset must have one row per FASTA file and 101 columns (100 embedding dimensions plus one temporal column)
- KMeans must be fit with exactly 3 centroids on the 100-dimensional embeddings, and the score computed on those same embeddings
- The example_1 data must be loaded from Data/cached/example_1/ and unpacked with built-in Python libraries (zipfile/tarfile) following MOVIS sanitizing conventions

## Limitations

- The score is only meaningful on the same embedding space used for clustering; changing Word2Vec dimensionality (paper chose 100-d empirically) invalidates comparison to the 0.495 reference
- MOVIS presupposes preprocessed (quality controlled and filtered) data; applying the clustering/score step to unfiltered data may not reproduce the reference
- The workflow is specific to the BWWTP Example 1 metaproteomics raw FASTA data; raw microarray-based data cannot be used directly as MOVIS input

## Evidence

- [results] K-Means clustering (3 centroids) of the embedded BWWTP metaproteomics raw FASTA data achieved a silhouette score of 0.495: [evidence span withheld — anchor status: not_found]
- [methods] a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns: [evidence span withheld — anchor status: not_found]
- [methods] It offers a way to filter a certain time-series period and remove one or more rows or columns: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a pickle object.: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
- [readme] The code is written in Python 3.8.11 and tested on Linux: "The code is written in Python 3.8.11 and tested on Linux with the following libraries installed"
