---
name: vega-lite-visualization-authoring
description: Use when a tabular time-series omic data set (CSV/TSV or pickled Pandas
  DataFrame with a temporal feature column) needs a reproducible, publication-ready
  chart exported as SVG, e.g. from Data/cached/example_1.
when_to_use_negative:
- Input is raw microarray-based data, which cannot be used directly as input without
  preprocessing (quality control and filtering)
- The analysis goal is direct integrative linking across omic-types (e.g. taxa linking
  for metagenomics), which MOVIS does not currently support
- The data lack an embedded temporal dimension and cannot be cast as time-series tabular
  data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0219
tools:
- name: Altair
  role: Declarative Vega-Lite chart authoring for time-series omic visualizations
  repo: https://github.com/AAnzel/MOVIS
- name: altair_saver
  role: Exporting the Altair chart to a publication-ready SVG file
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Importing CSV/TSV/pickled tabular data sets as Data Frame structures for plotting
- name: Streamlit
  role: Interactive graphical web interface through which visualizations are surfaced
  repo: https://github.com/AAnzel/MOVIS
- name: MOVIS
  role: Modular five-step analysis core whose visualization step hosts the authored
    charts
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:b55ff0d95d055a2151176a9376896c3a19237862f4d2ee8f04044b573c6f4113
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
  - altair_saver
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# vega-lite-visualization-authoring

## Summary

Authoring task-specific, publication-ready time-series visualizations with Altair 4.1.0 (Vega-Lite grammar) and altair_saver 0.5.0, as practiced in MOVIS for multi-omics data sets, with accessibility-aware color encoding and reproducible SVG export.

## When to use

Use when a tabular time-series omic data set (CSV/TSV or pickled Pandas DataFrame with a temporal feature column) needs a reproducible, publication-ready chart exported as SVG, e.g. from Data/cached/example_1.

## When NOT to use

- Input is raw microarray-based data, which cannot be used directly as input without preprocessing (quality control and filtering)
- The analysis goal is direct integrative linking across omic-types (e.g. taxa linking for metagenomics), which MOVIS does not currently support
- The data lack an embedded temporal dimension and cannot be cast as time-series tabular data

## Inputs

- Tabular time-series omic data set as CSV or TSV
- Pandas DataFrame stored as a pickle object (Data/cached/example_1)
- Temporal dimension present as a feature column

## Outputs

- Altair/Vega-Lite chart specification (reproducible across runs)
- SVG file, e.g. extension_omic_visualization.svg
- Task-specific, publication-ready visualization

## How to apply

Load the tabular input (CSV/TSV or pickled DataFrame from e.g. Data/cached/example_1) with Pandas 1.3.5, ensuring the temporal dimension is present as a feature column, and run data quality checks mirroring MOVIS's sanitizing step. Build the chart declaratively with Altair 4.1.0 (Vega-Lite), choosing encodings that respect accessibility considerations for color-blind users, and keep the generated Vega-Lite specification deterministic so repeated runs over the same input are bitwise-equivalent. Export via altair_saver 0.5.0 to a named SVG file. Rationale: MOVIS's five-step core (importing, sanitizing, filtering, analyzing, visualizing) treats the visualization step as responsible for visualizing and additional filtering, and the abstract states resulting visualizations are task-specific, reproducible, and publication-ready; verifying that the SVG exists, parses as valid SVG, and is reproducibly regenerated confirms correctness.

## Related tools

- **Altair** (Declarative Vega-Lite chart authoring for time-series omic visualizations) — https://github.com/AAnzel/MOVIS
- **altair_saver** (Exporting the Altair chart to a publication-ready SVG file) — https://github.com/AAnzel/MOVIS
- **Pandas** (Importing CSV/TSV/pickled tabular data sets as Data Frame structures for plotting)
- **Streamlit** (Interactive graphical web interface through which visualizations are surfaced) — https://github.com/AAnzel/MOVIS
- **MOVIS** (Modular five-step analysis core whose visualization step hosts the authored charts) — https://github.com/AAnzel/MOVIS

## Examples

```
import altair as alt, altair_saver; chart = alt.Chart(df).mark_line().encode(x='time', y='value', color='feature'); altair_saver.save(chart, 'extension_omic_visualization.svg', fmt='svg')
```

## Evaluation signals

- Exported SVG file exists on disk and parses as valid SVG (e.g. via an XML parser)
- Repeated runs over the same input data regenerate a bitwise-equivalent Vega-Lite specification
- Color encoding remains distinguishable for color-blind users (accessibility check)
- The visualization faithfully reflects the tabular time-series input, with the temporal feature column on the relevant axis after sanitizing/quality checks
- Output file name matches the task-specific target (e.g. extension_omic_visualization.svg)

## Limitations

- MOVIS presupposes data are preprocessed, i.e. quality controlled and filtered, before the core runs
- Raw microarray-based data cannot be used directly as input
- No taxa linking is enabled for metagenomics data, and direct integrative multi-omics linking is planned but not yet available
- Tool is pinned to specific versions (altair 4.1.0, altair_saver 0.5.0, pandas 1.3.5, Python 3.8.11, tested on Linux)

## Evidence

- [abstract] abstract: "The resulting visualizations are task-specific, reproducible, and publication-ready. MOVIS is built on open-source software and is easily extendable to accommodate different analytical tasks."
- [readme] readme: "The code is written in Python 3.8.11 and tested on Linux with the following libraries installed"
- [methods] methods: "The visualization step is responsible for visualizing and additional filtering of the data sets"
- [discussion] discussion: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
