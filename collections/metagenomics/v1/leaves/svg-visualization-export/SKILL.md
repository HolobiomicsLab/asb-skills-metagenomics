---
name: svg-visualization-export
description: Use when a tabular time-series omics dataset (CSV/TSV or pickled DataFrame
  with a temporal feature column) has passed importing, sanitizing, filtering, and
  analyzing steps and a publication-ready, reproducible figure must be produced and
  exported as SVG.
when_to_use_negative:
- The data is raw and unpreprocessed — the data analysis core presupposes quality-controlled
  and filtered input before visualization
- Raw microarray-based data is the input, since it cannot be used directly as input
  to MOVIS
- The goal is interactive web exploration rather than a static publication figure
  — the Streamlit interface handles interactive exploration instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: Altair
  role: Vega-Lite-based plotting library used to build the task-specific, color-blind-accessible
    time-series visualizations
- name: altair_saver
  role: Exports Altair charts to SVG files
- name: MOVIS
  role: Modular analysis core whose fifth sequential step (visualizing) hosts this
    export workflow
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Web interface framework through which the visualizations are presented in
    MOVIS
- name: Pandas
  role: DataFrame structures holding the imported tabular time-series data that feed
    the visualization step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:2e7f6f343b8810ec98398379cd8fd89920239d2eff7936d0a1ec8c3a7f984990
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# svg-visualization-export

## Summary

Generates task-specific, publication-ready visualizations of time-series multi-omics data with Altair (Vega-Lite) in MOVIS's visualizing core step, and exports them as SVG files for reproducible reuse.

## When to use

Use when a tabular time-series omics dataset (CSV/TSV or pickled DataFrame with a temporal feature column) has passed importing, sanitizing, filtering, and analyzing steps and a publication-ready, reproducible figure must be produced and exported as SVG.

## When NOT to use

- The data is raw and unpreprocessed — the data analysis core presupposes quality-controlled and filtered input before visualization
- Raw microarray-based data is the input, since it cannot be used directly as input to MOVIS
- The goal is interactive web exploration rather than a static publication figure — the Streamlit interface handles interactive exploration instead

## Inputs

- Tabular time-series omics dataset as CSV/TSV or pickled Pandas DataFrame with a temporal dimension as a feature column
- Analyzed/filtered Pandas DataFrame emerging from the sanitizing, filtering, and analyzing core steps

## Outputs

- SVG vector image file of the task-specific visualization (e.g., extension_omic_visualization.svg)
- Vega-Lite chart specification that is bitwise-equivalent across repeated runs on the same input

## How to apply

Position this skill at the fifth of MOVIS's five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing), where the visualization step is responsible for visualizing and additional filtering of the data sets. Build the figure with Altair 4.1.0 using Vega-Lite specifications, choosing color palettes that respect accessibility considerations for color-blind users, and render the plot tailored to the specific omic-type task (e.g., time-series line/heatmap views of BWWTP metagenomics, metaproteomics, metatranscriptomics, metabolomics, or physico-chemical data). Export the Altair chart via altair_saver 0.5.0 to an SVG file (e.g., extension_omic_visualization.svg) rather than raster formats so the output stays vector-based and publication-ready. Validate the output by checking the SVG file exists and parses as valid SVG, and confirm reproducibility by regenerating the Vega-Lite specification on repeated runs over the same input data and verifying it is bitwise-equivalent.

## Related tools

- **Altair** (Vega-Lite-based plotting library used to build the task-specific, color-blind-accessible time-series visualizations)
- **altair_saver** (Exports Altair charts to SVG files)
- **MOVIS** (Modular analysis core whose fifth sequential step (visualizing) hosts this export workflow) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Web interface framework through which the visualizations are presented in MOVIS)
- **Pandas** (DataFrame structures holding the imported tabular time-series data that feed the visualization step)

## Examples

```
import altair as alt; alt.Chart(df_marked_encoding='x=Time', 'y=Value').mark_line(); altair_saver.save(chart, 'extension_omic_visualization.svg', fmt='svg')
```

## Evaluation signals

- Exported SVG file exists at the expected path and parses as valid SVG (well-formed XML)
- Regenerating the visualization on the same input data yields a bitwise-equivalent Vega-Lite specification, confirming reproducibility
- Resulting figure is task-specific to the omic-type and publication-ready per the paper's criteria
- Color encoding passes color-blind accessibility considerations
- The visualization reflects only quality-controlled and filtered data, matching MOVIS's presupposition that input is preprocessed

## Limitations

- MOVIS presupposes data is preprocessed (quality controlled and filtered); visualizing raw data violates the core's assumptions
- Raw microarray-based data cannot be used directly as input
- No taxa linking is enabled for metagenomics data, limiting what certain multi-omic visualizations can express
- Color choices must respect accessibility considerations for color-blind users, constraining palette selection

## Evidence

- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [methods] The visualization step is responsible for visualizing and additional filtering of the data sets: "The visualization step is responsible for visualizing and additional filtering of the data sets"
- [abstract] The resulting visualizations are task-specific, reproducible, and publication-ready.: "The resulting visualizations are task-specific, reproducible, and publication-ready."
- [discussion] the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered.: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] altair|4.1.0 (dependency table): "|altair|4.1.0|
|altair_saver|0.5.0|"
