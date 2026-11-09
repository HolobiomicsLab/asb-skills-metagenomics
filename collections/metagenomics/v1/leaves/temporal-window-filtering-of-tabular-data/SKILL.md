---
name: temporal-window-filtering-of-tabular-data
description: Use when a tabular CSV/TSV dataset with a temporal feature has been imported
  and sanitized in MOVIS and the analysis targets a specific time-series period —
  e.g., isolating the early-April 2012 window of the BWWTP physico-chemical data to
  inspect the 29.62 °C temperature peak.
when_to_use_negative:
- Input is a non-tabular dataset (e.g., genomics/proteomics FASTA archives) — the
  MOVIS filtering step is present only for tabular data sets
- Data has not yet passed the MOVIS sanitizing step or lacks the required temporal
  information feature
- Raw, unpreprocessed inputs such as raw microarray-based data — the analysis core
  presupposes quality-controlled and filtered data, and such data cannot be used directly
  as input
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: MOVIS
  role: Web-based, modular tool whose filtering step restricts the tabular dataset
    to a time-series period and removes rows/columns before analysis and visualization
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Internal representation of the imported CSV/TSV data as Data Frame structures
    on which the filtering operates
- name: Streamlit
  role: Framework underlying MOVIS's interactive graphical web interface through which
    the filter controls are exposed
- name: Altair
  role: Rendering backend for the filtered time-series visualization canvas (e.g.,
    Time Heatmap / 'Feature through time')
- name: Python
  role: Implementation language of the MOVIS filtering and visualization pipeline
    (Python 3.8.11, tested on Linux)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:43381b5a810c8701e7b15677f29aa91f9db7a652124cce46119b918d8ee5a2ad
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# temporal-window-filtering-of-tabular-data

## Summary

Restrict a time-series tabular dataset (e.g., CSV/TSV loaded as a Pandas DataFrame in MOVIS) to a specific temporal window and optionally drop rows or columns, so downstream analysis or visualization focuses on the period of interest, such as the early-April 2012 temperature peak in the BWWTP physico-chemical data.

## When to use

Use when a tabular CSV/TSV dataset with a temporal feature has been imported and sanitized in MOVIS and the analysis targets a specific time-series period — e.g., isolating the early-April 2012 window of the BWWTP physico-chemical data to inspect the 29.62 °C temperature peak.

## When NOT to use

- Input is a non-tabular dataset (e.g., genomics/proteomics FASTA archives) — the MOVIS filtering step is present only for tabular data sets
- Data has not yet passed the MOVIS sanitizing step or lacks the required temporal information feature
- Raw, unpreprocessed inputs such as raw microarray-based data — the analysis core presupposes quality-controlled and filtered data, and such data cannot be used directly as input

## Inputs

- Tabular CSV or TSV time-series dataset with a temporal information feature (e.g., Data/cached/example_1 physico-chemical Processed data set 1 from the BWWTP data, 34 properties at a 2-h sampling rate)
- Pandas Data Frame structure as internally imported by MOVIS
- Target time window definition (start/end of the period of interest, e.g., beginning of April 2012)

## Outputs

- Filtered Pandas Data Frame restricted to the selected time-series period with unneeded rows/columns removed
- Time-restricted visualization (e.g., 'Feature through time' plot or Time Heatmap) exported as a lossless PNG

## How to apply

After loading the tabular dataset (MOVIS imports CSV/TSV as Pandas Data Frame structures) and running the sanitizing step — which performs various data quality checks and confirms the temporal information feature is present — apply the MOVIS filtering step, which is available only for tabular datasets. Use the filter controls to restrict the data to the time-series period of interest (e.g., beginning of April 2012 for the BWWTP Example 1 physico-chemical data) and to remove one or more unneeded rows or columns. The rationale is that the MOVIS data analysis core presupposes preprocessed, quality-controlled and filtered data, so temporal windowing must precede the analyzing and visualizing steps. Finally, generate the target visualization (e.g., 'Feature through time' or Time Heatmap) on the filtered series and export via the canvas export function as a lossless PNG. Validate the output against known reference values, such as the paper-reported rapid temperature increase peaking at 29.62 °C at the beginning of April 2012 in Processed data set 1.

## Related tools

- **MOVIS** (Web-based, modular tool whose filtering step restricts the tabular dataset to a time-series period and removes rows/columns before analysis and visualization) — https://github.com/AAnzel/MOVIS
- **Pandas** (Internal representation of the imported CSV/TSV data as Data Frame structures on which the filtering operates)
- **Streamlit** (Framework underlying MOVIS's interactive graphical web interface through which the filter controls are exposed)
- **Altair** (Rendering backend for the filtered time-series visualization canvas (e.g., Time Heatmap / 'Feature through time'))
- **Python** (Implementation language of the MOVIS filtering and visualization pipeline (Python 3.8.11, tested on Linux))

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest  # then load Data/cached/example_1 physico-chemical data, sanitize, filter to early April 2012
```

## Evaluation signals

- The sanitized dataset retains the required temporal information feature before filtering is attempted
- The filtered data frame contains only rows within the selected time window (e.g., early April 2012) and only the retained columns
- The exported visualization reproduces the paper-reported trend — for Processed data set 1, a rapid temperature increase peaking at 29.62 °C at the beginning of April 2012 (Fig. 8a Time Heatmap)
- The exported figure is a lossless PNG produced via the MOVIS canvas export function and matches the published case-study visualization under expert review
- Filtering occurs after importing/sanitizing and before the analyzing/visualizing core steps, consistent with MOVIS's five sequential core steps

## Limitations

- The filtering step is present only for tabular data sets — non-tabular inputs cannot be windowed this way
- The data analysis core presupposes preprocessed data; raw microarray-based data and similar inputs cannot be used directly
- Filtering alone does not substitute for the sanitizing step's data quality checks; both are prerequisites for downstream analysis

## Evidence

- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [methods] Filtering only for tabular datasets: "The filtering step is present only for tabular data sets"
- [methods] Filter a certain time-series period and remove rows/columns: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [results] Temperature peak finding for Processed data set 1: [evidence span withheld — anchor status: not_found]
- [methods] Tabular data imported as Pandas Data Frames: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [discussion] Preprocessing requirement before analysis: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] Example 1 data cached location in repository: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
