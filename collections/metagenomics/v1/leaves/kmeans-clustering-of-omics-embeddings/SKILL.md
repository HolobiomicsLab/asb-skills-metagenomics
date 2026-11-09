---
name: kmeans-clustering-of-omics-embeddings
description: Use when FASTA-derived omics data have been embedded into fixed-length
  numeric vectors (e.g., 100-d Word2Vec, 101-column table with a temporal column)
  and a distance-based clustering partition with a quantified quality score is needed.
when_to_use_negative:
- Input is already a feature/abundance table rather than archived FASTA files needing
  Word2Vec embedding
- Data have not been preprocessed — MOVIS's analysis core presupposes the data are
  quality controlled and filtered
- Topological/density-based structure is the goal and OPTICS is more appropriate than
  distance-based KMeans
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: MOVIS
  role: Modular time-series multi-omics exploration tool providing the sanitizing
    and data-clustering (KMeans/OPTICS) pipeline steps
  repo: https://github.com/AAnzel/MOVIS
- name: Gensim
  role: Trains the Word2Vec model that embeds each FASTA sequence into a 100-dimensional
    vector
- name: Biopython
  role: Parses FASTA sequences prior to embedding
- name: scikit-learn
  role: Provides KMeans (3 centroids) and silhouette_score for clustering and validation
- name: Pandas
  role: Holds the embedded vectors as a tabular DataFrame imported internally by MOVIS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:000b760a57dd2aabda73f3e910a2141a7e956ba72ebf7d6a6c91bf72ce150a53
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# kmeans-clustering-of-omics-embeddings

## Summary

Cluster Word2Vec-embedded time-series omics vectors with scikit-learn KMeans (as offered by MOVIS alongside OPTICS) and validate the partition via silhouette score, as demonstrated by reproducing a silhouette score of 0.495 on the Example 1 (BWWTP) metaproteomics Raw FASTA data with k=3 centroids.

## When to use

Use when FASTA-derived omics data have been embedded into fixed-length numeric vectors (e.g., 100-d Word2Vec, 101-column table with a temporal column) and a distance-based clustering partition with a quantified quality score is needed.

## When NOT to use

- Input is already a feature/abundance table rather than archived FASTA files needing Word2Vec embedding
- Data have not been preprocessed — MOVIS's analysis core presupposes the data are quality controlled and filtered
- Topological/density-based structure is the goal and OPTICS is more appropriate than distance-based KMeans

## Inputs

- Archived Raw FASTA files (zip/tar) from Data/cached/example_1/ (metaproteomics BWWTP data)
- Tabular embedding DataFrame: one row per FASTA file, 100 Word2Vec dimensions + 1 temporal column
- Number of KMeans centroids (k=3 for the reference reproduction)

## Outputs

- KMeans cluster assignment (3 centroids) for each embedded FASTA vector
- Single silhouette score value written to silhouette_score.txt (reference: 0.495)

## How to apply

Load the archived FASTA data (e.g., Data/cached/example_1/ from the Herold et al. BWWTP meta-omics study), unpacking with Python's zipfile/tarfile per MOVIS sanitizing conventions, and parse sequences with Biopython. Train a Gensim Word2Vec model to embed each sequence into a 100-dimensional vector, average sequence vectors per FASTA file to get one vector per file, and assemble a Pandas DataFrame with as many rows as FASTA files and 101 columns (100 embedding dimensions plus one temporal column). Fit scikit-learn KMeans with 3 centroids — MOVIS's distance-based clustering option alongside OPTICS — on the 100-dimensional embeddings. Compute the silhouette score of the assignment with sklearn's silhouette_score; success for the reference task is a score of 0.495 within numerical tolerance, written to silhouette_score.txt.

## Related tools

- **MOVIS** (Modular time-series multi-omics exploration tool providing the sanitizing and data-clustering (KMeans/OPTICS) pipeline steps) — https://github.com/AAnzel/MOVIS
- **Gensim** (Trains the Word2Vec model that embeds each FASTA sequence into a 100-dimensional vector)
- **Biopython** (Parses FASTA sequences prior to embedding)
- **scikit-learn** (Provides KMeans (3 centroids) and silhouette_score for clustering and validation)
- **Pandas** (Holds the embedded vectors as a tabular DataFrame imported internally by MOVIS)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import silhouette_score; labels = KMeans(n_clusters=3).fit_predict(X100d); score = silhouette_score(X100d, labels)
```

## Evaluation signals

- Computed silhouette score matches the reported reference value of 0.495 within a small numerical tolerance
- silhouette_score.txt exists and contains a single numeric score close to 0.495
- Embedded tabular dataset has exactly as many rows as FASTA files in the archived dataset and 101 columns (100 embedding dims + temporal)
- KMeans is run with 3 centroids on the 100-dimensional embeddings, matching the paper's configuration
- FASTA archive is unpacked with built-in Python libraries (zipfile/tarfile) following MOVIS sanitizing conventions

## Limitations

- MOVIS presupposes preprocessed input: 'the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered'
- Silhouette reproducibility depends on the empirically chosen 100-d Word2Vec embedding; other dimensions or random seeds may shift the score
- KMeans is one of only two distance-based clustering options MOVIS offers (alongside OPTICS), limiting algorithm choice for this step

## Evidence

- [results] K-Means clustering (3 centroids) of the embedded BWWTP metaproteomics raw FASTA data achieved a silhouette score of 0.495, indicating successful clustering.: [evidence span withheld — anchor status: not_found]
- [methods] Word2Vec embedding produces a tabular dataset with one row per FASTA file and 101 columns: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [methods] MOVIS offers data clustering as an optional pipeline step: "(4) optional data clustering, and (5) data set visualization"
- [readme] README documents the cached Example 1 data location: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
