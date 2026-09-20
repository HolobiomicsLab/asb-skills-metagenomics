---
name: movis-workflow-driven-data-exploration
description: Use when input is a tabular (CSV/TSV) time-series multi-omics data set
  (e.g., metabolomics over sampling timestamps) and the goal is step-wise exploration
  to detect temporal trends, anomalies, or substrate shifts and export publication-ready
  visualizations.
when_to_use_negative:
- The input is raw microarray-based data — such data cannot be used directly as input
  in MOVIS.
- The data is not yet quality controlled and filtered — the data analysis core presupposes
  preprocessed data, so MOVIS is the wrong first step for raw, uncleaned data.
- The task requires taxa linking for metagenomics data or direct integrative analysis
  across omics types — MOVIS is not explicitly designed for meta-omics integration,
  and no taxa linking is currently enabled.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: The web-based, modular tool implementing the five-step importing→sanitizing→filtering→analyzing→visualizing
    core used for time-series multi-omics exploration
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Framework powering MOVIS's interactive graphical web interface through which
    the workflow steps are executed
- name: Pandas
  role: Internally imports the CSV/TSV tabular data sets as Pandas Data Frame structures
    during the importing step
- name: Scikit-learn
  role: Provides the optional KMeans (distance-based) and OPTICS (density-based) clustering
    in the analyzing step
- name: Altair
  role: Renders the visualization canvas (Feature through time, Whisker plot, Correlation
    heatmap) with tooltip/brushing/zoom interactivity and PNG export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:0913066be5643d823b71f0a3901c8f62229b80412aa271079c129b6f1b64f1a4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# movis-workflow-driven-data-exploration

## Summary

Guides an agent through MOVIS's five-step sequential core (importing, sanitizing, filtering, analyzing, visualizing) to explore time-series multi-omics tabular data (CSV/TSV), e.g., reproducing the BWWTP Example 1 metabolomics substrate-shift analysis.

## When to use

Use when input is a tabular (CSV/TSV) time-series multi-omics data set (e.g., metabolomics over sampling timestamps) and the goal is step-wise exploration to detect temporal trends, anomalies, or substrate shifts and export publication-ready visualizations.

## When NOT to use

- The input is raw microarray-based data — such data cannot be used directly as input in MOVIS.
- The data is not yet quality controlled and filtered — the data analysis core presupposes preprocessed data, so MOVIS is the wrong first step for raw, uncleaned data.
- The task requires taxa linking for metagenomics data or direct integrative analysis across omics types — MOVIS is not explicitly designed for meta-omics integration, and no taxa linking is currently enabled.

## Inputs

- Tabular time-series multi-omics data set in CSV or TSV format (imported internally as a Pandas DataFrame)
- Built-in Example 1: BWWTP multi-meta-omics data (metagenomics, metaproteomics, metatranscriptomics, metabolomics, physico-chemical) stored in Data/cached/example_1 (raw or pickle)
- Processed data set 2: metabolomics-dominated composite data set (~95% metabolomics) from the BWWTP case study

## Outputs

- Publication-ready, task-specific, reproducible visualizations exported as PNG (e.g., Feature through time, Whisker plot, Correlation heatmap)
- Filtered time-series subset restricted to a user-specified time-series period
- Optional cluster assignments from KMeans (distance-based) or OPTICS (density-based) clustering

## How to apply

Launch MOVIS as a Streamlit web interface (Docker image aanzel/movis:latest, served on port 8501, or the hosted instance at movis.mathematik.uni-marburg.de). Follow the data analysis core's five sequential steps: importing (tabular CSV/TSV files are internally imported as Pandas Data Frame structures, or use the built-in Example 1 BWWTP data from Data/cached/example_1), sanitizing (various data quality checks including file-name validity checking and correction), filtering (available only for tabular data sets — restrict to a specific time-series period, e.g., November–December 2011, and remove rows/columns), analyzing (optionally apply KMeans distance-based or OPTICS density-based clustering via scikit-learn to the prepared time series), and visualizing (render Feature through time, Whisker plot, or Correlation heatmap views with tooltip/brushing/zoom interactivity, exported as PNG via Altair). Rationale: MOVIS presupposes quality-controlled, prefiltered input, so do not skip the sanitizing step; the filtering step is what enables isolating the temporal window where the discontinuity is expected.

## Related tools

- **MOVIS** (The web-based, modular tool implementing the five-step importing→sanitizing→filtering→analyzing→visualizing core used for time-series multi-omics exploration) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Framework powering MOVIS's interactive graphical web interface through which the workflow steps are executed)
- **Pandas** (Internally imports the CSV/TSV tabular data sets as Pandas Data Frame structures during the importing step)
- **Scikit-learn** (Provides the optional KMeans (distance-based) and OPTICS (density-based) clustering in the analyzing step)
- **Altair** (Renders the visualization canvas (Feature through time, Whisker plot, Correlation heatmap) with tooltip/brushing/zoom interactivity and PNG export)

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- Exported visualization shows a visible temporal trend discontinuity in the expected windows (e.g., early-to-mid November 2011 and early-to-mid December 2011 substrate shifts, with higher values in between and normalization after end of December 2011).
- The workflow completed all five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing) in order.
- The filtered data set contains only the requested time-series period and no removed rows/columns remain.
- Output figures are publication-ready and reproducible, matching the task-specific visualization choices (Feature through time, Whisker plot, or Correlation heatmap).

## Limitations

- Raw microarray-based data cannot be used directly as input.
- The data analysis core presupposes that the data is preprocessed (quality controlled and filtered).
- MOVIS is not explicitly designed to work with meta-omics data: no taxa linking is currently enabled, and the direct integrative aspect of multi-omics data is planned but not yet available.
- Further omics-wide, time-series-specific, and data-specific standards/guidelines are required.
- No changelog is available in the repository; the unstable Docker build may contain features that do not end up in the next release.

## Evidence

- [methods] five sequential core steps: [evidence span withheld — anchor status: not_found]
- [methods] filtering step scope: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [discussion] preprocessing precondition: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [results] Example 1 data set: [evidence span withheld — anchor status: not_found]
- [readme] Docker deployment: "docker pull aanzel/movis:latest
docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest"
- [methods] tabular import: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
