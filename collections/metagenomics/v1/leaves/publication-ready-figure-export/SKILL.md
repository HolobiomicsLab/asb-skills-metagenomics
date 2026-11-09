---
name: publication-ready-figure-export
description: Use when a time-series multi-omics dataset (e.g., BWWTP Example 1 metagenomics
  depth-of-coverage or physico-chemical Processed data set 1) has been explored in
  MOVIS and a reproducible, publication-quality figure must be produced and exported.
when_to_use_negative:
- Input is raw microarray-based data, which cannot be used directly as MOVIS input
- The dataset lacks the required embedded temporal information feature (sanitizing
  will not confirm it) or has not been preprocessed/quality-controlled, since the
  data analysis core presupposes quality-controlled, filtered data
- The task requires taxa linking for metagenomics data or direct multi-omics integration,
  which MOVIS does not support
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MOVIS
  role: Provides the modular pipeline (importing, sanitizing, filtering, analyzing,
    visualizing) whose visualization canvas performs the PNG export of time-series
    multi-omics figures
  repo: https://github.com/AAnzel/MOVIS
- name: Altair
  role: Rendering library (v4.1.0 with altair_saver 0.5.0) used by MOVIS to create
    the chart images that the canvas exports as PNG
- name: Streamlit
  role: Hosts the interactive graphical web interface (v1.5.1) through which the visualization
    canvas and its export function are accessed
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:95a338fc6f26d1f5a84cc8a003e770e0cda215490b98c441e1a91b4b5619e972
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.782608695652174
n_claims: 23
---

# publication-ready-figure-export

## Summary

Exports MOVIS time-series multi-omics visualizations (e.g., Whisker plot, Feature through time, Time Heatmap) via the MOVIS visualization canvas as lossless PNG files so that case-study figures are task-specific, reproducible, and publication-ready.

## When to use

Use when a time-series multi-omics dataset (e.g., BWWTP Example 1 metagenomics depth-of-coverage or physico-chemical Processed data set 1) has been explored in MOVIS and a reproducible, publication-quality figure must be produced and exported.

## When NOT to use

- Input is raw microarray-based data, which cannot be used directly as MOVIS input
- The dataset lacks the required embedded temporal information feature (sanitizing will not confirm it) or has not been preprocessed/quality-controlled, since the data analysis core presupposes quality-controlled, filtered data
- The task requires taxa linking for metagenomics data or direct multi-omics integration, which MOVIS does not support

## Inputs

- Tabular time-series dataset (CSV/TSV or pickle) from Data/cached/example_1/, e.g., Metagenomics Depth-of-coverage data or physico-chemical Processed data set 1 (34 properties, 2-h sampling rate)
- Running MOVIS instance (Docker container on port 8501 or online version)
- Selected MOVIS plot type from the nine supported time-series visualizations

## Outputs

- Lossless PNG file exported via the MOVIS visualization canvas
- Publication-ready, task-specific, reproducible visualization matching the corresponding published case-study figure

## How to apply

Run MOVIS locally (Docker image aanzel/movis:latest, published on port 8501) or via the online instance at movis.mathematik.uni-marburg.de, then load the target dataset — e.g., the Example 1 BWWTP metagenomics Depth-of-coverage data or physico-chemical Processed data set 1 stored under Data/cached/example_1/ in raw or pickled tabular form — so it is imported as a Pandas Data Frame (CSV/TSV). Pass the data through the MOVIS sanitizing step (data quality checks confirming the embedded temporal feature), and optionally apply the tabular-only filtering step to restrict the time-series period or drop rows/columns. In the visualization step, choose one of the nine supported plot types that exposes the pattern of interest (e.g., Whisker plot for the Q3/upper-limit sine-curve with ~one-month period in depth-of-coverage, Time Heatmap or Feature through time for the temperature peak of 29.62 °C in early April 2012). Render the chart on the MOVIS visualization canvas (built on Streamlit 1.5.1 with Altair 4.1.0 and altair_saver 0.5.0) and use the canvas export function to write a lossless PNG file. Judge correctness by expert comparison of the exported PNG against the corresponding published MOVIS case-study visualization.

## Related tools

- **MOVIS** (Provides the modular pipeline (importing, sanitizing, filtering, analyzing, visualizing) whose visualization canvas performs the PNG export of time-series multi-omics figures) — https://github.com/AAnzel/MOVIS
- **Altair** (Rendering library (v4.1.0 with altair_saver 0.5.0) used by MOVIS to create the chart images that the canvas exports as PNG)
- **Streamlit** (Hosts the interactive graphical web interface (v1.5.1) through which the visualization canvas and its export function are accessed)

## Examples

```
docker pull aanzel/movis:latest; docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- The exported file is a lossless PNG produced by the MOVIS visualization canvas export function
- Expert review confirms the exported figure reproduces the reported case-study pattern — e.g., Q3 and upper-limit values tracing a sine-curve shape with a roughly one-month period (with slight discrepancy around early November 2011) for depth-of-coverage, or the temperature peak of 29.62 °C at the beginning of April 2012 for Processed data set 1
- Re-running the same dataset, plot type, and filters in MOVIS yields a visually identical figure (reproducibility of the published visualization)
- The plot type used is one of the nine supported MOVIS time-series visualizations (e.g., Whisker plot, Feature through time, Time Heatmap)
- The figure is derived from a dataset that passed the sanitizing step's data quality checks confirming the temporal feature

## Limitations

- The filtering step is present only for tabular data sets, so non-tabular inputs cannot be period-restricted before visualization
- Specific data such as raw microarray-based data cannot be used directly as input; data must be preprocessed (quality controlled and filtered) before the data analysis core can use it
- MOVIS is not explicitly designed to work with meta-omics data — no taxa linking is enabled and direct integrative multi-omics aspects are planned but not yet available

## Evidence

- [abstract] The resulting visualizations are task-specific, reproducible, and publication-ready.: "The resulting visualizations are task-specific, reproducible, and publication-ready."
- [methods] The visualization step is responsible for visualizing and additional filtering of the data sets: "The visualization step is responsible for visualizing and additional filtering of the data sets"
- [methods] It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [readme] It is stored at Data/cached/example_1/ in either a raw format or as a pickle object.: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
