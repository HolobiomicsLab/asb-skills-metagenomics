---
name: multi-omics-cross-tabulation-analysis
description: Use when a k-means clustering of time-series multi-omics samples (e.g.,
  Example 1 BWWTP metaproteomics FASTA embeddings) needs verification that a cluster
  label correlates with a temporal sub-period of the sampling time series.
when_to_use_negative:
- The data set has not been preprocessed — MOVIS presupposes data is already quality
  controlled and filtered before the analysis core
- Input is raw unclustered data (e.g., raw Metaproteomics Raw FASTA files) — clustering
  (analyzing step) must precede the cross-tabulation
- The goal is direct integrative linking across omic types (e.g., taxa linking for
  metagenomics) — MOVIS does not currently support this
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3892
- http://edamontology.org/topic_3520
tools:
- name: Pandas
  role: Import the clustered tabular data set as a DataFrame and build the cluster-by-time
    cross-tabulation via groupby/aggregation
- name: scikit-learn
  role: Source of the K-Means (k=3) clustering whose labels are cross-tabulated against
    the sampling period
- name: MOVIS
  role: Produces the clustered time-series multi-omics data set via its importing,
    sanitizing, filtering, analyzing, and visualizing pipeline on Example 1 BWWTP
    data
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:53221b118839b2fe945dcf64f644a7ec975cbf0f537c3b5a92be0944ba174c27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - MOVIS
  - Pandas
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3892
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multi-omics-cross-tabulation-analysis

## Summary

Builds a cross-tabulation of K-Means cluster membership against a temporal feature (e.g., early vs. later sampling period) for time-series multi-omics data sets, using Pandas aggregation to quantify whether a cluster is predominantly associated with a specific part of the sampling period.

## When to use

Use when a k-means clustering of time-series multi-omics samples (e.g., Example 1 BWWTP metaproteomics FASTA embeddings) needs verification that a cluster label correlates with a temporal sub-period of the sampling time series.

## When NOT to use

- The data set has not been preprocessed — MOVIS presupposes data is already quality controlled and filtered before the analysis core
- Input is raw unclustered data (e.g., raw Metaproteomics Raw FASTA files) — clustering (analyzing step) must precede the cross-tabulation
- The goal is direct integrative linking across omic types (e.g., taxa linking for metagenomics) — MOVIS does not currently support this

## Inputs

- Clustered Example 1 (BWWTP) metaproteomics data set (e.g., CSV/TSV imported as a Pandas DataFrame) with one row per sample
- K-Means (k=3) cluster label column per sample
- Temporal feature column indicating the sampling period (early vs. later)

## Outputs

- Cross-tabulation of cluster membership vs. sampling period
- Fraction of class-1 samples mapping to the later sampling period
- Summary table cluster_time_verification.csv

## How to apply

Load the clustered data set — e.g., the Example 1 (BWWTP) metaproteomics data, where one row per sample carries the K-Means (k=3) cluster label and a temporal feature column — with Pandas 1.3.5 as a Pandas Data Frame. Group samples by cluster label and by sampling period (early vs. later) to construct a cross-tabulation of cluster membership against time. Quantify the fraction of class-1 samples falling into the later sampling period via Pandas aggregation. Validate against the reported pattern that class-1 (rectangle-marked) samples in the k=3 K-Means clustering came predominantly from the later time of the sampling period; write the summary table to cluster_time_verification.csv. The clustering itself derives from the analyzing step of MOVIS's five-step core (importing, sanitizing, filtering, analyzing, visualizing), where data must be preprocessed/quality-controlled first.

## Related tools

- **Pandas** (Import the clustered tabular data set as a DataFrame and build the cluster-by-time cross-tabulation via groupby/aggregation)
- **scikit-learn** (Source of the K-Means (k=3) clustering whose labels are cross-tabulated against the sampling period)
- **MOVIS** (Produces the clustered time-series multi-omics data set via its importing, sanitizing, filtering, analyzing, and visualizing pipeline on Example 1 BWWTP data) — https://github.com/AAnzel/MOVIS

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('Data/cached/example_1/metaproteomics_clustered.csv'); ct = pd.crosstab(df['cluster'], df['period']); frac = (ct.loc[1, 'later'] / ct.loc[1].sum()); print(frac); ct.to_csv('cluster_time_verification.csv')"
```

## Evaluation signals

- Majority (>50%) of class-1 samples map to the later sampling period, matching the reported k=3 K-Means result
- The cross-tabulation row totals equal the number of samples per cluster label (no samples dropped during grouping)
- Output cluster_time_verification.csv contains one fraction per cluster-period combination with fractions summing to 1 within each cluster
- Input DataFrame row count matches the number of samples in the Example 1 metaproteomics data set as loaded by MOVIS

## Limitations

- Data must be preprocessed (quality controlled and filtered) before analysis — raw microarray-based data cannot be used directly
- MOVIS brings each omic-type into equal participation but does not directly integrate multi-omics; taxa linking is not enabled for metagenomics data
- Cluster labels from K-Means are arbitrary; class-1 identity depends on reproducing the same k=3 clustering of the Example 1 metaproteomics FASTA embeddings

## Evidence

- [results] Samples clustered in the class-1 (rectangles) came in majority from the later time of the sampling period in the k=3 K-Means clustering of the metaproteomics embeddings.: [evidence span withheld — anchor status: not_found]
- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [discussion] the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered.: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object.: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
