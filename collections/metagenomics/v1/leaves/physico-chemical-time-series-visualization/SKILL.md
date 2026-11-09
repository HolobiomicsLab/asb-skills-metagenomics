---
name: physico-chemical-time-series-visualization
description: Use when a tabular (CSV/TSV) time-series physico-chemical dataset must
  be explored in MOVIS to detect temporal trends, anomalies, or maxima in a single
  feature — e.g., locating when a temperature series reaches its peak value.
when_to_use_negative:
- Input is raw microarray-based data — the article notes such data 'cannot be used
  directly as input' and must first be preprocessed (quality controlled and filtered)
- The goal is taxa linking or direct integrative analysis of metagenomics data — MOVIS
  is not explicitly designed for meta-omics integration and 'no taxa linking is currently
  enabled'
- The dataset is FASTA-based (raw genomics/proteomics archives) rather than tabular
  — the filtering step is present only for tabular data sets
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Web-based modular tool whose five-step core (importing, sanitizing, filtering,
    analyzing, visualizing) renders the time-series feature visualization
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Internally imports the tabular CSV/TSV dataset as Pandas DataFrame structures
- name: Altair
  role: Rendering backend (Altair/Vega-Lite) for the time-series visualizations on
    the MOVIS canvas
- name: Streamlit
  role: Framework underlying MOVIS's interactive graphical web interface through which
    filtering and visualization are operated
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:50d9fe98611267c023329e9ea2239690548b41d212ee3acbb1ed689db658fd36
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# physico-chemical-time-series-visualization

## Summary

Visualize a physico-chemical feature (e.g., temperature) from a tabular time-series multi-omics dataset in MOVIS — demonstrated on BWWTP Example 1 'Processed data set 1' (34 physico-chemical properties, 2-h sampling rate), where a Time Heatmap reveals a temperature peak of 29.62 °C at the beginning of April 2012.

## When to use

Use when a tabular (CSV/TSV) time-series physico-chemical dataset must be explored in MOVIS to detect temporal trends, anomalies, or maxima in a single feature — e.g., locating when a temperature series reaches its peak value.

## When NOT to use

- Input is raw microarray-based data — the article notes such data 'cannot be used directly as input' and must first be preprocessed (quality controlled and filtered)
- The goal is taxa linking or direct integrative analysis of metagenomics data — MOVIS is not explicitly designed for meta-omics integration and 'no taxa linking is currently enabled'
- The dataset is FASTA-based (raw genomics/proteomics archives) rather than tabular — the filtering step is present only for tabular data sets

## Inputs

- Tabular time-series dataset in CSV or TSV format with a temporal feature (e.g., BWWTP 'Processed data set 1' with 34 physico-chemical properties at 2-h sampling rate, stored in Data/cached/example_1/ as raw files or pickle)
- A named physico-chemical feature to plot (e.g., 'T C' / temperature)

## Outputs

- A 'Feature through time' or Time Heatmap visualization rendered on the MOVIS canvas (Altair/Vega-Lite)
- Exported publication-ready PNG file (SVG or Vega-Lite specification as alternates)

## How to apply

Load the target dataset (e.g., the built-in Example 1 BWWTP physico-chemical 'Processed data set 1' from Data/cached/example_1, available raw or as a pickle) into MOVIS, which imports CSV/TSV tabular data as Pandas DataFrames. Run MOVIS's sanitizing step, which performs data quality checks and confirms the temporal dimension is present as a feature. Apply the filtering step (available only for tabular datasets) to restrict to the time-series period of interest and optionally drop unneeded rows/columns. Render a 'Feature through time' or Time Heatmap visualization on the visualization canvas (Altair/Vega-Lite, color-blind-accessible). Export the figure via the canvas export function as a lossless PNG (SVG or Vega-Lite spec as alternates) and compare it against the reference finding — e.g., the rapid temperature increase peaking at 29.62 °C in early April 2012.

## Related tools

- **MOVIS** (Web-based modular tool whose five-step core (importing, sanitizing, filtering, analyzing, visualizing) renders the time-series feature visualization) — https://github.com/AAnzel/MOVIS
- **Pandas** (Internally imports the tabular CSV/TSV dataset as Pandas DataFrame structures)
- **Altair** (Rendering backend (Altair/Vega-Lite) for the time-series visualizations on the MOVIS canvas)
- **Streamlit** (Framework underlying MOVIS's interactive graphical web interface through which filtering and visualization are operated)

## Examples

```
docker pull aanzel/movis:latest; docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- All five core MOVIS steps (importing, sanitizing, filtering, analyzing, visualizing) complete without error on the tabular physico-chemical dataset
- The sanitizing step confirms the temporal information feature is present in the dataset before visualization
- The exported figure reproduces the paper-reported trend: a rapid temperature increase peaking at 29.62 °C at the beginning of April 2012 for Processed data set 1 (expert review against the published Fig. 8a Time Heatmap)
- The exported file is a lossless PNG (or SVG/Vega-Lite spec) and matches the source data's temporal dimension

## Limitations

- Raw microarray-based data cannot be used directly as input; data must be preprocessed (quality controlled and filtered) beforehand
- MOVIS is not explicitly designed to work with meta-omics data — no taxa linking is currently enabled, and direct multi-omics integration is planned but not yet available
- The filtering step is present only for tabular data sets, so the full filter-then-visualize workflow does not apply to raw FASTA inputs

## Evidence

- [results] The selected data set contains 34 different physico-chemical properties with a 2-h sampling rate.: "The selected data set contains 34 different physico-chemical properties with a 2-h sampling rate."
- [results] Visualizing the BWWTP physico-chemical Processed data set 1 in MOVIS (Time Heatmap, Fig. 8a) shows a rapid temperature increase that peaks at the beginning of April 2012, reaching as high as 29.62 °C.: [evidence span withheld — anchor status: not_found]
- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [readme] It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object.: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
