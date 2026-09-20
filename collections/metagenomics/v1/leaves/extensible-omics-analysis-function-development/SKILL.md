---
name: extensible-omics-analysis-function-development
description: Use when a tabular time-series omic data set does not fit any of MOVIS's
  five supported omic-types and you need to add omic-specific functionality to the
  analysis core rather than preprocess the data externally.
when_to_use_negative:
- The input is raw microarray-based data, which cannot be used directly as input to
  the MOVIS analysis core
- The data is not yet preprocessed — the core presupposes quality-controlled and filtered
  data
- The goal is direct integrative (cross-omic taxa linking) analysis, which is planned
  but not yet available in MOVIS
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3077
tools:
- name: MOVIS
  role: Modular time-series multi-omics exploration tool whose five-step analysis
    core is extended with the new omic-specific high-level function
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Imports the tabular CSV/TSV time-series data set as a Pandas DataFrame with
    the temporal dimension as a feature column
- name: Altair
  role: Generates the task-specific Vega-Lite time-series visualization with color-blind-accessible
    styling, exported to SVG
- name: Streamlit
  role: Framework underlying MOVIS's interactive web interface into which the extended
    function integrates
- name: Python
  role: Implementation language of the extension (MOVIS code is written in Python
    3.8.11)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:2a80c175b1883172415b820c3e77a7879162a8c07df6a8f3edb64270b10491c2
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
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# extensible-omics-analysis-function-development

## Summary

A method for extending the MOVIS modular time-series multi-omics analysis core with a single high-level function that supports an additional omic-type, yielding task-specific, reproducible, publication-ready visualizations. It leverages MOVIS's five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing) and its open-source, easily-extendable design.

## When to use

Use when a tabular time-series omic data set does not fit any of MOVIS's five supported omic-types and you need to add omic-specific functionality to the analysis core rather than preprocess the data externally.

## When NOT to use

- The input is raw microarray-based data, which cannot be used directly as input to the MOVIS analysis core
- The data is not yet preprocessed — the core presupposes quality-controlled and filtered data
- The goal is direct integrative (cross-omic taxa linking) analysis, which is planned but not yet available in MOVIS

## Inputs

- Tabular time-series omic data set (CSV, TSV, or pickled Pandas DataFrame) from Data/cached/example_1 with the temporal dimension present as a feature column
- MOVIS source code (Source/ directory, e.g. main.py, common.py, visualize.py) describing the modular analysis core

## Outputs

- A new high-level core function supporting the additional omic-type
- extension_omic_visualization.svg — a task-specific, publication-ready, color-blind-accessible time-series visualization exported from Altair/Vega-Lite

## How to apply

Load the tabular time-series data from Data/cached/example_1 (CSV/TSV or pickled DataFrame) with Pandas, ensuring the temporal dimension is present as a feature column, mirroring MOVIS's convention that tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures. Run the sanitizing step of data quality checks — including validity of file names and their correction if they do not adhere to naming rules — matching the core's quality-control conventions. Implement the new high-level function inside the data analysis core so it wraps the five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing), respecting that the filtering step applies only to tabular data sets (filter a certain time-series period, remove rows or columns). Generate the task-specific visualization with Altair (Vega-Lite), e.g. version 4.1.0 as pinned in the README, and respect accessibility considerations for color-blind users. Export the visualization as an SVG via altair_saver (0.5.0) named extension_omic_visualization.svg, and validate reproducibility by confirming a bitwise-equivalent Vega-Lite specification on repeated runs.

## Related tools

- **MOVIS** (Modular time-series multi-omics exploration tool whose five-step analysis core is extended with the new omic-specific high-level function) — https://github.com/AAnzel/MOVIS
- **Pandas** (Imports the tabular CSV/TSV time-series data set as a Pandas DataFrame with the temporal dimension as a feature column)
- **Altair** (Generates the task-specific Vega-Lite time-series visualization with color-blind-accessible styling, exported to SVG)
- **Streamlit** (Framework underlying MOVIS's interactive web interface into which the extended function integrates)
- **Python** (Implementation language of the extension (MOVIS code is written in Python 3.8.11))

## Examples

```
import pandas as pd; df = pd.read_pickle('Data/cached/example_1/data.pkl'); chart = alt.Chart(df).mark_line().encode(x='time', y='value', color=alt.Color('feature', scale=alt.Scale(scheme='colorblind'))); altair_saver.save(chart, 'extension_omic_visualization.svg')
```

## Evaluation signals

- extension_omic_visualization.svg exists and parses as valid SVG
- The Vega-Lite specification regenerated on repeated runs over the same input data is bitwise-equivalent, confirming reproducibility
- The sanitizing step reports passing data quality checks and valid file names before analysis
- The visualization is color-blind-accessible and task-specific, consistent with MOVIS's publication-ready visualization claims
- The new function composes the five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing) rather than bypassing them

## Limitations

- Raw microarray-based data cannot be used directly as input; preprocessing is required
- The data analysis core presupposes that the data is already quality controlled and filtered
- No taxa linking is enabled for metagenomics data, and direct integrative multi-omics analysis is planned but not yet available

## Evidence

- [methods] five-step core rationale: [evidence span withheld — anchor status: not_found]
- [abstract] extensibility finding: "MOVIS is built on open-source software and is easily extendable to accommodate different analytical tasks."
- [methods] sanitizing checks: "the sanitizing step consists of various data quality checks"
- [methods] tabular import convention: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [readme] publication-ready output claim: "The resulting visualizations are task-specific, reproducible, and publication-ready."
