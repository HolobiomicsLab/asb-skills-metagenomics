---
name: metaproteomics-cluster-label-analysis
description: Use when validating or reproducing a clustering result on time-series
  metaproteomics data (e.g., MOVIS Example 1 embeddings) by cross-tabulating cluster
  labels against sampling-time periods and testing whether a majority of one cluster
  maps to a specific temporal segment.
when_to_use_negative:
- Input is raw metaproteomics FASTA or unclustered tabular data — the workflow presupposes
  data that is already 'quality controlled and filtered' and clustered (k=3 labels
  present)
- The data is not a time-series multi-omics dataset with a temporal/sampling-period
  dimension — the verification tests cluster-vs-time composition and has no meaning
  without a temporal column
- You need direct multi-omics integration or taxa linking for metagenomics data —
  the paper states 'no taxa linking is currently enabled' and integrative analysis
  is 'planned but not yet directly available in MOVIS'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: Pandas
  role: Imports the clustered CSV/TSV as a DataFrame and performs the groupby/crosstab
    aggregation of cluster labels against sampling period
- name: scikit-learn
  role: Provides the K-Means algorithm (k=3) used to generate the cluster labels on
    the 101-column FASTA embeddings before temporal verification
- name: MOVIS
  role: Source of the Example 1 metaproteomics workflow (Raw FASTA selection, Word2Vec
    embedding, optional data clustering) whose class-1 output this skill verifies
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:2f7debfb477a287ac3103d3de2c9faa1bf4e284c9c4bb98de8e8118c4c1ae4d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - MOVIS
  - Pandas
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metaproteomics-cluster-label-analysis

## Summary

Reproduces the temporal composition check of K-Means (k=3) cluster labels computed on the Example 1 (BWWTP) metaproteomics FASTA embeddings, verifying the reported finding that class-1 samples came predominantly from the later part of the sampling period.

## When to use

Use when validating or reproducing a clustering result on time-series metaproteomics data (e.g., MOVIS Example 1 embeddings) by cross-tabulating cluster labels against sampling-time periods and testing whether a majority of one cluster maps to a specific temporal segment.

## When NOT to use

- Input is raw metaproteomics FASTA or unclustered tabular data — the workflow presupposes data that is already 'quality controlled and filtered' and clustered (k=3 labels present)
- The data is not a time-series multi-omics dataset with a temporal/sampling-period dimension — the verification tests cluster-vs-time composition and has no meaning without a temporal column
- You need direct multi-omics integration or taxa linking for metagenomics data — the paper states 'no taxa linking is currently enabled' and integrative analysis is 'planned but not yet directly available in MOVIS'

## Inputs

- Clustered Example 1 (BWWTP) metaproteomics dataset: one row per sample with K-Means (k=3) cluster label and a temporal/sampling-period feature column (CSV/TSV or pickle from Data/cached/example_1/)
- Word2Vec FASTA embeddings of the metaproteomics Raw FASTA files (101 columns per FASTA file) if re-clustering is required

## Outputs

- cluster_time_verification.csv: cross-tabulation of cluster label vs. sampling period with the class-1 later-period fraction
- Pass/fail verification result (majority >50% of class-1 samples in the later sampling period)

## How to apply

Load the clustered Example 1 metaproteomics dataset (one row per sample, with the K-Means k=3 cluster label and a temporal feature column) using pandas, which the paper states 'internally imported as Pandas Data Frame structures'. Build a cross-tabulation of cluster membership (0, 1, 2) against sampling period (early vs. later) with pandas aggregation, then compute the fraction of class-1 samples falling in the later sampling period. The decision threshold from the reported finding is strict: verification succeeds only if >50% of class-1 (rectangle-marked) samples map to the later sampling period. Save the aggregated summary as cluster_time_verification.csv. The underlying embeddings derive from MOVIS's Word2Vec step on Raw FASTA files, which yields 'a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns'.

## Related tools

- **Pandas** (Imports the clustered CSV/TSV as a DataFrame and performs the groupby/crosstab aggregation of cluster labels against sampling period)
- **scikit-learn** (Provides the K-Means algorithm (k=3) used to generate the cluster labels on the 101-column FASTA embeddings before temporal verification)
- **MOVIS** (Source of the Example 1 metaproteomics workflow (Raw FASTA selection, Word2Vec embedding, optional data clustering) whose class-1 output this skill verifies) — https://github.com/AAnzel/MOVIS

## Examples

```
import pandas as pd; df = pd.read_pickle('Data/cached/example_1/metaproteomics_clustered.pkl'); ct = pd.crosstab(df['cluster'], df['sampling_period']); (ct.loc[1, 'later'] / ct.loc[1].sum() > 0.5); ct.to_csv('cluster_time_verification.csv')
```

## Evaluation signals

- The cross-tabulation covers all three clusters (k=3 labels) and both sampling periods with no dropped samples (row/column sums equal total sample count)
- The class-1 later-period fraction is strictly greater than 0.5, matching the reported pattern that class-1 samples came 'in majority from the later time of the sampling period'
- cluster_time_verification.csv is produced and its shape matches the number of clusters x number of temporal categories
- When re-embedding is needed, the embedding table has 101 columns and one row per FASTA file, consistent with the Word2Vec output described in the paper
- Cluster label vocabulary matches the source data (class-1 identified with rectangles in the MOVIS visualization)

## Limitations

- MOVIS 'is not explicitly designed to work with meta-omics data, no taxa linking is currently enabled', so cluster labels cannot be traced to taxonomy
- The analysis core 'presupposes that the data is preprocessed, that is to say, quality controlled and filtered'; raw or unsanitized FASTA/tabular input will not reproduce the result
- Cluster label indices (e.g., which cluster is 'class-1') can vary between K-Means runs; the class-1 identity must be anchored to the reported rectangle-marked group rather than assumed by index
- Direct integrative multi-omics analysis is 'planned but not yet directly available in MOVIS', so this verification is per-omic (metaproteomics only)

## Evidence

- [results] Samples clustered in the class-1 (rectangles) came in majority from the later time of the sampling period in the k=3 K-Means clustering of the metaproteomics embeddings.: [evidence span withheld — anchor status: not_found]
- [methods] a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [results] we selected Metaproteomics data, and then Raw FASTA files.: "we selected Metaproteomics data, and then Raw FASTA files."
- [readme] It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object.: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
