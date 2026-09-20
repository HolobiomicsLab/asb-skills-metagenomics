---
name: time-series-clustering-with-kmeans-and-optics
description: Use when a quality-controlled, filtered tabular time-series multi-omics
  dataset (CSV/TSV loaded into MOVIS) should be grouped by temporal behavior to expose
  substrate shifts or trends, e.g. the November–December 2011 metabolomics window.
when_to_use_negative:
- The dataset is raw or unpreprocessed — the data analysis core presupposes the data
  is quality controlled and filtered, so raw microarray-based data or unfiltered imports
  cannot be used directly
- The input is non-tabular (e.g. archived FASTA files), since the filtering step —
  which scopes the time window for clustering — is present only for tabular data sets
- The goal is direct integrative linking across omics types (e.g. taxa linking for
  metagenomics), which MOVIS does not currently support
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Web-based tool exposing the optional data-clustering step (KMeans or OPTICS)
    within its five-step analysis core and visualizing the clustered time series
  repo: https://github.com/AAnzel/MOVIS
- name: scikit-learn
  role: Underlying library (v1.0.2) implementing the KMeans and OPTICS clustering
    algorithms invoked through MOVIS's analysis core
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Framework (v1.5.1) of the interactive graphical web interface through which
    clustering options are selected and run
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Holds the tabular CSV/TSV dataset as a DataFrame structure that the clustering
    step operates on
- name: Altair
  role: Renders the clustered time-series visualizations (with tooltip/brushing/zooming)
    and supports PNG export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:1c5c0c7ef0b24f2897afbc706239c53675f6cd03f21cfdf1d3c1ec6794933d39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Altair
  - MOVIS
  - Pandas
  - Python
  - Streamlit
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Time-series clustering with KMeans and OPTICS

## Summary

Apply MOVIS's optional clustering step — KMeans (distance-based) or OPTICS (density-based) — to a prepared time-series multi-omics tabular dataset to group temporally similar features before visualization, using the BWWTP Example 1 metabolomics data as the reference case.

## When to use

Use when a quality-controlled, filtered tabular time-series multi-omics dataset (CSV/TSV loaded into MOVIS) should be grouped by temporal behavior to expose substrate shifts or trends, e.g. the November–December 2011 metabolomics window.

## When NOT to use

- The dataset is raw or unpreprocessed — the data analysis core presupposes the data is quality controlled and filtered, so raw microarray-based data or unfiltered imports cannot be used directly
- The input is non-tabular (e.g. archived FASTA files), since the filtering step — which scopes the time window for clustering — is present only for tabular data sets
- The goal is direct integrative linking across omics types (e.g. taxa linking for metagenomics), which MOVIS does not currently support

## Inputs

- Tabular time-series multi-omics dataset (CSV or TSV) imported as a Pandas DataFrame in MOVIS
- Sanitized (data-quality-checked) dataset output from MOVIS's sanitizing step
- Time-period-filtered dataset (e.g. Example 1 / Processed data set 2, metabolomics from the BWWTP, restricted to November–December 2011)

## Outputs

- Cluster assignments for time-series features (KMeans distance-based or OPTICS density-based groupings)
- Clustered visualization rendered on the MOVIS visualization canvas (Feature through time, Whisker plot, Correlation heatmap)
- Exported publication-ready PNG figure showing cluster-separated temporal trends

## How to apply

First ensure the dataset has passed MOVIS's importing, sanitizing, and filtering steps: the data analysis core presupposes preprocessed data, i.e. quality controlled and filtered, so clustering must never be run on raw imports. In the Streamlit web interface, choose clustering from the analysis core's optional step 4 (data clustering): KMeans if distance-based grouping of time-series features is desired, or OPTICS if density-based structure with non-uniform cluster shapes is expected. Restrict the time-series period first (e.g. the November–December 2011 window of Processed data set 2, metabolomics from the BWWTP) so clusters reflect the interval of interest rather than diluting it with normalized periods. After clustering, proceed to step 5 (data set visualization) to render clusters on the visualization canvas (e.g. Feature through time, Whisker plot, Correlation heatmap) and export as PNG. Judge success by whether clusters separate the temporal windows showing the reported substrate-shift signal from the baseline period.

## Related tools

- **MOVIS** (Web-based tool exposing the optional data-clustering step (KMeans or OPTICS) within its five-step analysis core and visualizing the clustered time series) — https://github.com/AAnzel/MOVIS
- **scikit-learn** (Underlying library (v1.0.2) implementing the KMeans and OPTICS clustering algorithms invoked through MOVIS's analysis core) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Framework (v1.5.1) of the interactive graphical web interface through which clustering options are selected and run) — https://github.com/AAnzel/MOVIS
- **Pandas** (Holds the tabular CSV/TSV dataset as a DataFrame structure that the clustering step operates on)
- **Altair** (Renders the clustered time-series visualizations (with tooltip/brushing/zooming) and supports PNG export)

## Examples

```
docker pull aanzel/movis:latest; docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest  # then select KMeans/OPTICS in the Streamlit UI at http://localhost:8501/
```

## Evaluation signals

- Confirm the dataset passed the sanitizing step's data quality checks before clustering (no quality-controlled violations remain)
- Confirm the input was restricted via the filtering step to the intended time-series period (e.g. November–December 2011), since filtering is only available for tabular data
- The clustered visualization reproduces the expected temporal discontinuity: substrate shifts from early to mid-November 2011 and from early to mid-December 2011, with higher values in between and normalization after end of December 2011
- The exported PNG is task-specific, reproducible, and publication-ready, matching the paper's reproducibility claim for the integrated Example 1 data set
- Re-running the same KMeans/OPTICS options on the same filtered data yields consistent cluster groupings across sessions

## Limitations

- The data analysis core presupposes that the data is preprocessed, i.e. quality controlled and filtered; clustering on unprepared data is out of scope
- The filtering step preceding clustering is present only for tabular data sets, so clustering of FASTA-derived stages cannot be time-window filtered in the same way
- MOVIS is not explicitly designed to work with meta-omics data — no taxa linking is enabled, and direct integrative multi-omics aspects are planned but not yet available
- The code was written and tested only in Python 3.8.11 on Linux with pinned library versions (e.g. scikit-learn 1.0.2, pandas 1.3.5, streamlit 1.5.1); other environments may behave differently

## Evidence

- [methods] The data analysis step is the central and most intricate part of the data analysis core: "The data analysis step is the central and most intricate part of the data analysis core"
- [methods] optional data clustering within the five presentation steps: "(4) optional data clustering, and (5) data set visualization"
- [discussion] clustering requires preprocessed data: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [methods] filtering scopes tabular data before analysis: "The filtering step is present only for tabular data sets"
- [other] KMeans and OPTICS clustering options in MOVIS workflow: [evidence span withheld — anchor status: not_found]
