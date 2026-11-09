---
name: omics-clustering-dimensionality-reduction
description: Use when a filtered Pandas DataFrame time-series omics table needs unsupervised
  grouping via distance-based or density-based clustering plus embedding, before rendering
  an interactive time-series visualization.
when_to_use_negative:
- The data has not been preprocessed — the analysis core presupposes that the data
  is quality controlled and filtered before use
- The input is raw microarray-based data, which cannot be used directly as input
- The input is a raw (non-tabular) FASTA archive that still needs importing and embedding
  into a tabular structure first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MOVIS
  role: Web-based, modular Streamlit application whose five-step data analysis core
    performs the importing, sanitizing, filtering, analyzing, and visualizing sequence;
    clustering and embedding occur in its analyzing step
  repo: https://github.com/AAnzel/MOVIS
- name: Scikit-learn
  role: Supplies the KMeans (distance-based) and OPTICS (density-based) clustering
    algorithms and PCA, MDS, and t-SNE embeddings used in the analyzing step
- name: Pandas
  role: Represents the imported tabular time-series data as DataFrames that are filtered
    before clustering and embedding
- name: Altair
  role: Renders the clustered/embedded time-series visualizations on the visualization
    canvas in a color-blind-accessible design
- name: Streamlit
  role: Hosts the interactive graphical web interface through which the analysis workflow
    steps are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:16919c4270073f98359d3349838a4ccea2c52355505ef9537bfa649ba317e6f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Altair
  - MOVIS
  - Pandas
  - Python
  - Scikit-learn
  - Streamlit
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# omics-clustering-dimensionality-reduction

## Summary

Applies optional clustering (KMeans or OPTICS) and low-dimensional projection (PCA, MDS, or t-SNE) to filtered time-series multi-omics tabular data within MOVIS's analyzing step, so that temporal omics structure (e.g., physico-chemical properties sampled at a 2-h rate from the BWWTP dataset) can be explored as cluster-colored, publication-ready visualizations.

## When to use

Use when a filtered Pandas DataFrame time-series omics table needs unsupervised grouping via distance-based or density-based clustering plus embedding, before rendering an interactive time-series visualization.

## When NOT to use

- The data has not been preprocessed — the analysis core presupposes that the data is quality controlled and filtered before use
- The input is raw microarray-based data, which cannot be used directly as input
- The input is a raw (non-tabular) FASTA archive that still needs importing and embedding into a tabular structure first

## Inputs

- Filtered time-series tabular omics data as a Pandas DataFrame (CSV/TSV imported, e.g., 34 physico-chemical properties from the BWWTP Example 1 dataset)
- Optionally, genomics/proteomics FASTA-derived tabular data (e.g., Word2Vec-embedded tables with 101 columns, or 43-feature proteomics physico-chemical tables)
- User-selected clustering method (KMeans or OPTICS) and embedding method (PCA, MDS, or t-SNE)

## Outputs

- Cluster-labeled 2D projection of the filtered time-series omics data
- Cluster visualization rendered with Altair/Vega-Lite on the MOVIS canvas
- Exportable publication-ready PNG (alternates: SVG or Vega-Lite specification)

## How to apply

First ensure the data has passed MOVIS's importing (CSV/TSV loaded as Pandas Data Frames) and sanitizing steps, since the analysis core presupposes quality-controlled data. In the analyzing step — described as 'the central and most intricate part of the data analysis core' — optionally cluster the filtered DataFrame using KMeans for distance-based grouping or OPTICS for density-based grouping (both from scikit-learn 1.0.2), then project the data with PCA, MDS, or t-SNE to 2D for cluster visualization. Choose KMeans when cluster shape assumptions are reasonable; choose OPTICS when clusters of varying density are expected. Feed the clustered/embedded result into the visualizing step rendered with Altair/Vega-Lite on Streamlit, and export as PNG (SVG or Vega-Lite spec as alternates).

## Related tools

- **MOVIS** (Web-based, modular Streamlit application whose five-step data analysis core performs the importing, sanitizing, filtering, analyzing, and visualizing sequence; clustering and embedding occur in its analyzing step) — https://github.com/AAnzel/MOVIS
- **Scikit-learn** (Supplies the KMeans (distance-based) and OPTICS (density-based) clustering algorithms and PCA, MDS, and t-SNE embeddings used in the analyzing step)
- **Pandas** (Represents the imported tabular time-series data as DataFrames that are filtered before clustering and embedding)
- **Altair** (Renders the clustered/embedded time-series visualizations on the visualization canvas in a color-blind-accessible design)
- **Streamlit** (Hosts the interactive graphical web interface through which the analysis workflow steps are executed)

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- All five core steps (importing, sanitizing, filtering, analyzing, visualizing) complete without error on the chosen dataset
- A rendered, exportable visualization matching the source data's temporal dimension is produced
- The exported PNG (or SVG/Vega-Lite spec) reproduces the cluster structure shown in the interactive Altair canvas
- The input DataFrame is a filtered, quality-controlled Pandas table with a temporal feature present before clustering
- Embedding output reduces to 2 dimensions suitable for the cluster-colored visualization

## Limitations

- Since MOVIS is not explicitly designed to work with meta-omics data, no taxa linking is currently enabled
- Direct integrative aspect of multi-omics data is planned but not yet directly available in MOVIS
- Raw microarray-based data cannot be used directly as input; data must be preprocessed (quality controlled and filtered) before use
- Clustering is optional — the analyzing step may be run with embedding only if clustering is not desired

## Evidence

- [methods] core five steps: [evidence span withheld — anchor status: not_found]
- [methods] central analysis step: "The data analysis step is the central and most intricate part of the data analysis core"
- [methods] filtering scope: "The filtering step is present only for tabular data sets"
- [methods] pandas import: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [results] dataset detail: "The selected data set contains 34 different physico-chemical properties with a 2-h sampling rate."
- [discussion] preprocessing requirement: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [methods] embedding output shape: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [readme] run instructions: "docker pull aanzel/movis:latest
docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest"
