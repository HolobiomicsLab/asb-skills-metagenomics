---
name: temporal-multi-omics-data-handling
description: Use when input is archived time-series multi-omics data (e.g. FASTA archives
  under Data/cached/example_1/) and the goal is to embed sequences and cluster them
  (KMeans or OPTICS) in the MOVIS pipeline.
when_to_use_negative:
- Input is already a preprocessed feature table rather than archived FASTA — the analysis
  core presupposes data is quality controlled and filtered first
- Raw microarray-based data cannot be used directly as input in MOVIS
- Taxa linking or direct integrative multi-omics linking is required — MOVIS does
  not support taxa linking for metagenomics data and direct integration is planned
  but not yet available
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Modular web-based (Streamlit) tool providing the five-step core (importing,
    sanitizing, filtering, analyzing, visualizing) in which this data handling is
    executed
  repo: https://github.com/AAnzel/MOVIS
- name: Gensim
  role: Word2Vec training to embed FASTA sequences into 100-dimensional vectors
- name: Biopython
  role: Parsing each FASTA sequence prior to embedding
- name: scikit-learn
  role: KMeans clustering (3 centroids) and silhouette_score computation on the embeddings
- name: Pandas
  role: Internal tabular storage of the embedded dataset as a Pandas DataFrame
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:164419209907e6ebdbd779aa776d8055125cfec9a4ffe36c37477a4ebbb6be0c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# temporal-multi-omics-data-handling

## Summary

Method for importing, sanitizing, embedding, and clustering archived time-series multi-omics data (e.g. the Herold et al. BWWTP Example 1 metaproteomics Raw FASTA files) within MOVIS's five-step core (importing, sanitizing, filtering, analyzing, visualizing) to reproduce analyses such as the 0.495 silhouette score.

## When to use

Use when input is archived time-series multi-omics data (e.g. FASTA archives under Data/cached/example_1/) and the goal is to embed sequences and cluster them (KMeans or OPTICS) in the MOVIS pipeline.

## When NOT to use

- Input is already a preprocessed feature table rather than archived FASTA — the analysis core presupposes data is quality controlled and filtered first
- Raw microarray-based data cannot be used directly as input in MOVIS
- Taxa linking or direct integrative multi-omics linking is required — MOVIS does not support taxa linking for metagenomics data and direct integration is planned but not yet available

## Inputs

- Archived FASTA files (raw or pickle) of time-series multi-omics data, e.g. Data/cached/example_1/ from Herold et al. BWWTP
- Tabular data sets (CSV or TSV) for the filtering-capable branches of the pipeline

## Outputs

- Pandas DataFrame with one row per FASTA file and 101 columns (100 Word2Vec embedding dimensions + 1 temporal column)
- KMeans cluster assignment per FASTA file (e.g. 3 centroids)
- silhouette_score.txt containing a single numeric silhouette score (reference: 0.495)

## How to apply

Load the archived data set (e.g. Example 1 BWWTP metaproteomics Raw FASTA from Data/cached/example_1/) and unpack it with built-in Python libraries (zipfile/tarfile) following MOVIS's sanitizing conventions (file name validity checks, data quality checks). Parse each FASTA sequence with Biopython and train a Gensim Word2Vec model to embed sequences into 100-dimensional vectors, averaging per FASTA file to yield a Pandas DataFrame with one row per file and 101 columns (100 embedding dimensions plus one temporal column). Run the MOVIS analysis core's optional data clustering step with scikit-learn KMeans (e.g. 3 centroids, the distance-based option alongside OPTICS). Judge success by computing scikit-learn's silhouette_score on the embeddings — for Example 1 metaproteomics with k=3 the reference value is 0.495 within a small numerical tolerance, written to silhouette_score.txt.

## Related tools

- **MOVIS** (Modular web-based (Streamlit) tool providing the five-step core (importing, sanitizing, filtering, analyzing, visualizing) in which this data handling is executed) — https://github.com/AAnzel/MOVIS
- **Gensim** (Word2Vec training to embed FASTA sequences into 100-dimensional vectors)
- **Biopython** (Parsing each FASTA sequence prior to embedding)
- **scikit-learn** (KMeans clustering (3 centroids) and silhouette_score computation on the embeddings)
- **Pandas** (Internal tabular storage of the embedded dataset as a Pandas DataFrame)

## Examples

```
from gensim.models import Word2Vec; from sklearn.cluster import KMeans; from sklearn.metrics import silhouette_score; km = KMeans(n_clusters=3).fit(X); score = silhouette_score(X, km.labels_)
```

## Evaluation signals

- Computed silhouette score matches the reference 0.495 for Example 1 metaproteomics Raw FASTA with Word2Vec 100-d embedding and KMeans k=3 within a small numerical tolerance
- silhouette_score.txt exists and contains a numeric value close to 0.495
- Embedded tabular dataset has exactly one row per FASTA file and 101 columns (100 embedding dimensions + temporal column)
- Archive unpacking follows MOVIS sanitizing conventions (file name validity checked/corrected, data quality checks passed)
- Cluster count equals the specified centroid number (k=3) and every FASTA file receives an assignment

## Limitations

- MOVIS is not explicitly designed to work with meta-omics data, so no taxa linking is currently enabled for metagenomics
- Direct integrative aspect of multi-omics data is planned but not yet directly available in MOVIS
- Raw microarray-based data cannot be used directly as input; the analysis core presupposes preprocessed (quality controlled and filtered) data

## Evidence

- [methods] five sequential core steps: [evidence span withheld — anchor status: not_found]
- [methods] archived FASTA importing: "In the first core step, i.e., importing, genomics, and proteomics data sets can be provided as archived FASTA files"
- [methods] Word2Vec tabular dataset shape: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [results] Raw FASTA selection: "we selected Metaproteomics data, and then Raw FASTA files."
- [discussion] preprocessing presupposition: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] Example 1 data location: "The data used in the **Example 1** comes from the following paper"
- [readme] environment: "The code is written in Python 3.8.11 and tested on Linux with the following libraries installed"
