---
name: bwwtp-multi-omics-dataset-loading
description: Use when a task requires loading the built-in Example 1 BWWTP time-series
  multi-meta-omics data (metagenomics, metaproteomics, metatranscriptomics, metabolomics,
  physico-chemical) into MOVIS for downstream sanitizing, filtering, analysis, or
  visualization.
when_to_use_negative:
- Input is raw microarray-based data, which cannot be used directly as MOVIS input
- Data are not preprocessed — the analysis core presupposes quality-controlled and
  filtered data
- Task requires taxa linking across metagenomics data or direct multi-omics integration,
  which MOVIS does not currently support
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3077
tools:
- name: MOVIS
  role: Web-based modular tool whose importing step loads the cached Example 1 BWWTP
    tabular datasets as Pandas DataFrames
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas 1.3.5
  role: Library into whose Data Frame structures tabular CSV/TSV datasets are internally
    imported
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit 1.5.1
  role: Framework underlying MOVIS's interactive graphical web interface used to select
    and load the Example 1 dataset
  repo: https://github.com/AAnzel/MOVIS
- name: Python 3.8.11
  role: Language in which MOVIS is written; pickle-serialized DataFrames in Data/cached/example_1
    are Python objects
  repo: https://github.com/AAnzel/MOVIS
- name: Docker
  role: Deployment mechanism via the aanzel/movis image to run MOVIS locally on port
    8501
  repo: https://hub.docker.com/r/aanzel/movis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:8f491ba184ebb2df9e873b7300fb0bd13e60a6dec9cb7611a771918f40ada7f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Altair
  - Docker
  - MOVIS
  - Pandas
  - Pandas 1.3.5
  - Python
  - Python 3.8.11
  - Streamlit
  - Streamlit 1.5.1
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# bwwtp-multi-omics-dataset-loading

## Summary

Load the BWWTP multi-meta-omics Example 1 data (including Processed data set 1 physico-chemical, 34 properties at 2-h sampling rate) into MOVIS from Data/cached/example_1, where tabular CSV/TSV files are imported as Pandas DataFrames, as the entry step for time-series exploration and visualization.

## When to use

Use when a task requires loading the built-in Example 1 BWWTP time-series multi-meta-omics data (metagenomics, metaproteomics, metatranscriptomics, metabolomics, physico-chemical) into MOVIS for downstream sanitizing, filtering, analysis, or visualization.

## When NOT to use

- Input is raw microarray-based data, which cannot be used directly as MOVIS input
- Data are not preprocessed — the analysis core presupposes quality-controlled and filtered data
- Task requires taxa linking across metagenomics data or direct multi-omics integration, which MOVIS does not currently support

## Inputs

- Data/cached/example_1/ BWWTP Example 1 dataset (raw CSV/TSV or pickle objects) from Herold et al. 2020
- Processed data set 1 physico-chemical table: 34 properties, 2-h sampling rate, with temporal information feature and temperature (T C) column
- Composite multi-omics tables (metagenomics, metaproteomics, metatranscriptomics, metabolomics), e.g. Processed data set 2 metabolomics

## Outputs

- Pandas Data Frame structures of the imported tabular BWWTP sub-datasets inside MOVIS
- Sanitized dataset confirmed to contain the required temporal information feature, ready for the MOVIS filtering step

## How to apply

Start MOVIS (Docker image aanzel/movis:latest published on port 8501, or the online instance at movis.mathematik.uni-marburg.de) and select the built-in Example 1 page. The Example 1 data originate from the Herold et al. 2020 Nature Communications BWWTP study and are cached at Data/cached/example_1/ in raw CSV/TSV or pickled Pandas DataFrame form; tabular files are internally imported as Pandas DataFrames (Pandas 1.3.5) by MOVIS's importing core step. Choose the required sub-dataset — for physico-chemical work select Processed data set 1, which contains 34 different physico-chemical properties with a 2-h sampling rate, including the temperature (T C) feature. After import, run MOVIS's sanitizing step, which performs various data quality checks (including file-name validity checking) and confirms the required temporal information feature is present before filtering or analysis. Rationale: MOVIS's analysis core presupposes data that are quality controlled and filtered, and the importing step is the first of five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing).

## Related tools

- **MOVIS** (Web-based modular tool whose importing step loads the cached Example 1 BWWTP tabular datasets as Pandas DataFrames) — https://github.com/AAnzel/MOVIS
- **Pandas 1.3.5** (Library into whose Data Frame structures tabular CSV/TSV datasets are internally imported) — https://github.com/AAnzel/MOVIS
- **Streamlit 1.5.1** (Framework underlying MOVIS's interactive graphical web interface used to select and load the Example 1 dataset) — https://github.com/AAnzel/MOVIS
- **Python 3.8.11** (Language in which MOVIS is written; pickle-serialized DataFrames in Data/cached/example_1 are Python objects) — https://github.com/AAnzel/MOVIS
- **Docker** (Deployment mechanism via the aanzel/movis image to run MOVIS locally on port 8501) — https://hub.docker.com/r/aanzel/movis

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- The imported Processed data set 1 exposes 34 different physico-chemical properties sampled at a 2-h rate
- The MOVIS sanitizing step completes its various data quality checks without reporting a missing temporal information feature
- The dataset loads identically whether read from raw CSV/TSV or the cached pickle object in Data/cached/example_1/
- Downstream MOVIS filtering/visualization (e.g. Time Heatmap of temperature) can run on the imported frame, reproducing the paper's Fig. 8a trend of a rapid increase peaking at 29.62 °C at the beginning of April 2012

## Limitations

- MOVIS is not explicitly designed to work with meta-omics data, so no taxa linking is currently enabled for metagenomics
- Direct integrative aspects of multi-omics data are planned but not yet available in MOVIS
- Specific data such as raw microarray-based data cannot be used directly as input; data must be preprocessed (quality controlled and filtered) before the analysis core

## Evidence

- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing: [evidence span withheld — anchor status: not_found]
- [results] We focused on the built-in Example 1 [29] ... metagenomics, metaproteomics, metatranscriptomics, metabolomics, and physico-chemical data from the Biological WasteWater Treatment Plant (BWWTP): "metagenomics, metaproteomics, metatranscriptomics, metabolomics, and physico-chemical data from the Biological WasteWater Treatment Plant (BWWTP)"
- [readme] It is stored at Data/cached/example_1/ in either a raw format or as a pickle object.: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
- [discussion] the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered.: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
