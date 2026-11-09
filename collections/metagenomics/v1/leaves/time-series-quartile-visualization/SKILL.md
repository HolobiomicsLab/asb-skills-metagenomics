---
name: time-series-quartile-visualization
description: Use when a tabular (CSV/TSV) dataset has an embedded temporal feature
  and the goal is to detect periodic or envelope-level patterns in the population
  distribution — e.g., quartile traces cycling with ~one-month periodicity.
when_to_use_negative:
- The data is raw microarray-based data, which cannot be used directly as MOVIS input
  and must first be preprocessed (quality controlled and filtered).
- Taxa linking across metagenomics features is required — MOVIS is not explicitly
  designed for meta-omics data and no taxa linking is enabled.
- Direct integrative cross-omics analysis is the goal — that aspect is planned but
  not yet directly available in MOVIS.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: MOVIS
  role: Web-based, modular time-series multi-omics exploration tool whose data analysis
    core (importing, sanitizing, filtering, analyzing, visualizing) renders the Whisker
    plot of quartile envelopes
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Framework underlying the interactive graphical web interface through which
    the dataset is imported and visualized
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Internally imports the tabular CSV/TSV dataset as Pandas DataFrame structures
    for filtering and visualization
- name: Altair
  role: Plotting library backing the Whisker plot rendering and the tooltip/brushing/zooming
    interactivity on the visualization canvas
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:2dbc0fd1cc1e985bc3b793533152c4d9c124e53530b8ad886c82980024c80018
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# time-series-quartile-visualization

## Summary

Visualize a tabular time-series multi-omics dataset in MOVIS using the Whisker plot to expose quartile envelopes (Q3 and upper limit) of the distribution over time, revealing periodic temporal patterns such as the ~one-month sine-curve in the BWWTP metagenomics depth-of-coverage data.

## When to use

Use when a tabular (CSV/TSV) dataset has an embedded temporal feature and the goal is to detect periodic or envelope-level patterns in the population distribution — e.g., quartile traces cycling with ~one-month periodicity.

## When NOT to use

- The data is raw microarray-based data, which cannot be used directly as MOVIS input and must first be preprocessed (quality controlled and filtered).
- Taxa linking across metagenomics features is required — MOVIS is not explicitly designed for meta-omics data and no taxa linking is enabled.
- Direct integrative cross-omics analysis is the goal — that aspect is planned but not yet directly available in MOVIS.

## Inputs

- Tabular CSV/TSV or pickled time-series dataset from Data/cached/example_1/ (Metagenomics Depth-of-coverage, Example 1 BWWTP)
- Dataset with an embedded temporal feature (dates spanning e.g. 2011 for BWWTP data)

## Outputs

- MOVIS Whisker plot visualization exposing Q3 and upper-limit quartile envelopes over time
- Exported PNG file of the visualization from the MOVIS visualization canvas

## How to apply

Load the tabular dataset (raw or pickled) from Data/cached/example_1/ — for this skill the Example 1 (BWWTP) Metagenomics Depth-of-coverage data from Herold et al. (Nat Commun 11, 5281, 2020). Import it into the MOVIS data analysis core (Streamlit 1.5.1 web interface, internally a Pandas 1.3.5 DataFrame); the sanitizing step runs data quality checks and confirms the embedded temporal feature. Optionally use the tabular-only filtering step to restrict the time-series period or remove rows/columns, then select the Whisker plot from the nine supported visualization types (built on Altair 4.1.0) to render quartile envelopes. Export the figure as a PNG from the visualization canvas. Rationale: whisker/quartile statistics of the depth-of-coverage series expose the population-level envelope that single-feature time plots would obscure.

## Related tools

- **MOVIS** (Web-based, modular time-series multi-omics exploration tool whose data analysis core (importing, sanitizing, filtering, analyzing, visualizing) renders the Whisker plot of quartile envelopes) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Framework underlying the interactive graphical web interface through which the dataset is imported and visualized) — https://github.com/AAnzel/MOVIS
- **Pandas** (Internally imports the tabular CSV/TSV dataset as Pandas DataFrame structures for filtering and visualization)
- **Altair** (Plotting library backing the Whisker plot rendering and the tooltip/brushing/zooming interactivity on the visualization canvas)

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest  # then open http://localhost:8501/, import Data/cached/example_1 Depth-of-coverage, render Whisker plot, export PNG
```

## Evaluation signals

- The sanitizing step completes data quality checks and confirms the embedded temporal feature without errors.
- The Whisker plot shows the third quartile and upper limit tracing a sine-curve-like pattern with a period of roughly one month.
- Any discrepancy in the sine-curve pattern appears around the beginning of November 2011 and is attributable to increased outliers.
- The exported PNG reproduces the corresponding reported MOVIS visualization for Example 1 (figure-level comparison).

## Limitations

- MOVIS presupposes preprocessed data: 'the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered.'
- No taxa linking is enabled for metagenomics data, and direct integrative multi-omics analysis is planned but not yet available.
- Filtering is available only for tabular data sets; non-tabular inputs cannot be period-restricted this way.

## Evidence

- [results] the third quartile and upper limit values trace a sine-curve shape with a period of roughly one month: [evidence span withheld — anchor status: not_found]
- [methods] the filtering step is present only for tabular data sets: "The filtering step is present only for tabular data sets"
- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [methods] the sanitizing step consists of various data quality checks: "the sanitizing step consists of various data quality checks"
- [readme] two time-series multi-omics data sets have been integrated and successfully reproduced: "As of the time of writing, two time-series multi-omics data sets have been integrated and successfully reproduced. The resulting visualizations are task-specific, reproducible, and publication-ready."
