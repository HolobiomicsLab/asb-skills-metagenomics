---
name: metagenomics-depth-of-coverage-import
description: Use when a tabular metagenomics depth-of-coverage time series (Example
  1 / BWWTP data) must be loaded into MOVIS for time-series visualization, and the
  temporal feature must be confirmed during sanitizing.
when_to_use_negative:
- Input is raw microarray-based data, which cannot be used directly as MOVIS input
  and must be preprocessed first
- Data has not been quality controlled and filtered, since the data analysis core
  presupposes preprocessed input
- The goal is taxa linking across metagenomics samples, which MOVIS does not currently
  enable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Web-based, modular tool whose analysis core imports, sanitizes, filters, analyzes,
    and visualizes the depth-of-coverage tabular time series
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit 1.5.1
  role: Open-source framework providing the interactive graphical web interface through
    which the dataset is imported
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas 1.3.5
  role: Internally imports the CSV/TSV depth-of-coverage data as Pandas Data Frame
    structures
- name: Altair 4.1.0
  role: Rendering library behind the MOVIS visualization canvas exporting the Whisker
    plot as PNG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:3c14a2952a8fceda33a4d0df207e3ecb01af4c7bc5ed57d04cf3c6aee5ccdd01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Altair
  - Altair 4.1.0
  - MOVIS
  - Pandas
  - Pandas 1.3.5
  - Python
  - Streamlit
  - Streamlit 1.5.1
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metagenomics-depth-of-coverage-import

## Summary

Import the Herold et al. BWWTP metagenomics Depth-of-coverage tabular dataset from Data/cached/example_1/ (raw or pickled form) into the MOVIS data analysis core so that the embedded temporal feature passes sanitizing and the series can be visualized (e.g., Whisker plot) to expose monthly periodicity in Q3 and upper-limit values.

## When to use

Use when a tabular metagenomics depth-of-coverage time series (Example 1 / BWWTP data) must be loaded into MOVIS for time-series visualization, and the temporal feature must be confirmed during sanitizing.

## When NOT to use

- Input is raw microarray-based data, which cannot be used directly as MOVIS input and must be preprocessed first
- Data has not been quality controlled and filtered, since the data analysis core presupposes preprocessed input
- The goal is taxa linking across metagenomics samples, which MOVIS does not currently enable

## Inputs

- Metagenomics Depth-of-coverage tabular dataset from Data/cached/example_1/ (raw CSV/TSV or Python pickle)
- Example 1 (BWWTP multi-metaomics) directory hierarchy with Metagenomics -> Depth-of-coverage selection

## Outputs

- Sanitized Pandas DataFrame with confirmed embedded temporal feature inside the MOVIS analysis core
- Whisker plot / Feature through time visualization exposing Q3 and upper-limit envelopes
- PNG export of the visualization from the MOVIS visualization canvas

## How to apply

Load the Example 1 metagenomics Depth-of-coverage dataset from Data/cached/example_1/ in either raw tabular or pickled form; MOVIS (Streamlit 1.5.1 + Pandas 1.3.5) imports CSV/TSV tabular data as Pandas Data Frame structures. In the importing step, navigate Metagenomics -> Depth-of-coverage, which presents a directory hierarchy of the underlying dataset. Let the sanitizing step run its data quality checks to confirm the embedded temporal feature and validate file names against naming rules. Optionally apply the tabular-only filtering step to restrict a time-series period or remove rows/columns. Then select a time-series visualization (Whisker plot or Feature through time) from the nine supported MOVIS plot types and export the figure via the visualization canvas as a PNG. Rationale: the depth-of-coverage series carries a temporal dimension that the core presupposes, so correct import is a precondition for reproducing the reported ~one-month sine-curve pattern in Q3 and upper limits.

## Related tools

- **MOVIS** (Web-based, modular tool whose analysis core imports, sanitizes, filters, analyzes, and visualizes the depth-of-coverage tabular time series) — https://github.com/AAnzel/MOVIS
- **Streamlit 1.5.1** (Open-source framework providing the interactive graphical web interface through which the dataset is imported) — https://github.com/AAnzel/MOVIS
- **Pandas 1.3.5** (Internally imports the CSV/TSV depth-of-coverage data as Pandas Data Frame structures)
- **Altair 4.1.0** (Rendering library behind the MOVIS visualization canvas exporting the Whisker plot as PNG)

## Examples

```
docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest  # then open http://localhost:8501/ and load Example 1 Metagenomics -> Depth-of-coverage
```

## Evaluation signals

- Sanitizing step reports the embedded temporal feature in the imported depth-of-coverage table and passes data quality checks
- The Whisker plot shows the third quartile and upper limit values tracing a sine-curve shape with a period of roughly one month
- Any slight discrepancy appears around the beginning of November 2011, caused by increased outliers — matching the reported MOVIS visualization for Example 1
- Exported PNG from the visualization canvas reproduces the corresponding reported MOVIS figure for Example 1

## Limitations

- MOVIS is not explicitly designed to work with meta-omics data, so no taxa linking is currently enabled for metagenomics data
- Data must be preprocessed (quality controlled and filtered) before the data analysis core can be used
- Direct integrative aspects of multi-omics data are planned but not yet available in MOVIS

## Evidence

- [results] Depth-of-coverage import path: "we selected Metagenomics data, and then Depth-of-coverage, which presented us with a directory hierarchy of the underlying data set."
- [methods] Sanitizing checks temporal feature: "the sanitizing step consists of various data quality checks"
- [methods] Tabular import as DataFrame: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [methods] Tabular-only filtering: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [readme] Data location in repo: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
