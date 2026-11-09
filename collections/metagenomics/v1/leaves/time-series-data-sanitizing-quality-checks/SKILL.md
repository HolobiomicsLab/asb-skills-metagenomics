---
name: time-series-data-sanitizing-quality-checks
description: Use when tabular CSV/TSV time-series data (e.g., the BWWTP physico-chemical
  Processed data set 1 loaded into MOVIS as a Pandas Data Frame) must be quality-checked
  and names/structures validated before downstream filtering or visualization.
when_to_use_negative:
- The data analysis core presupposes that the data is already preprocessed, i.e.,
  quality controlled and filtered — do not rely on sanitizing to fix raw microarray-based
  data, which cannot be used directly as input
- Taxa linking for metagenomics data is not supported by MOVIS, so sanitizing will
  not enable meta-omics taxa integration
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3077
tools:
- name: MOVIS
  role: Web-based, modular time-series multi-omics exploration tool whose sanitizing
    core step performs the data quality checks
  repo: https://github.com/AAnzel/MOVIS
- name: Python
  role: Implementation language of the MOVIS code (written in Python 3.8.11) that
    executes the sanitizing logic
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Open-source framework (v1.5.1) underlying MOVIS's interactive web interface
    through which the sanitizing step is invoked
- name: Pandas
  role: Library (v1.3.5) providing the Data Frame structures into which tabular CSV/TSV
    data sets are imported prior to sanitizing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:7b62d6610cbf5fce83807a985ed2fc9acd5fa872753e051b5768d692e35db9aa
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# time-series-data-sanitizing-quality-checks

## Summary

Run the MOVIS sanitizing step on imported tabular time-series data sets (e.g., the BWWTP physico-chemical Processed data set 1) to perform data quality checks — including file name validity checking and correction — before filtering, analysis, and visualization.

## When to use

Use when tabular CSV/TSV time-series data (e.g., the BWWTP physico-chemical Processed data set 1 loaded into MOVIS as a Pandas Data Frame) must be quality-checked and names/structures validated before downstream filtering or visualization.

## When NOT to use

- The data analysis core presupposes that the data is already preprocessed, i.e., quality controlled and filtered — do not rely on sanitizing to fix raw microarray-based data, which cannot be used directly as input
- Taxa linking for metagenomics data is not supported by MOVIS, so sanitizing will not enable meta-omics taxa integration

## Inputs

- Tabular CSV or TSV time-series data set (e.g., Processed data set 1, physico-chemical BWWTP data with 34 properties at a 2-h sampling rate, from Data/cached/example_1)
- Archived FASTA files (for genomics/proteomics data sets, imported in the first core step)

## Outputs

- Sanitized, quality-checked data set with valid file names and a confirmed temporal information feature, ready for the MOVIS filtering step

## How to apply

After importing the tabular data set (CSV/TSV files are internally imported as Pandas Data Frame structures in MOVIS, built on Streamlit 1.5.1 and Pandas 1.3.5), execute the sanitizing step, which consists of various data quality checks. These include checking the validity of the file names and correcting them if they do not adhere to MOVIS's naming rules, and confirming that the required temporal information feature is present in the tabular data set (as demonstrated when reproducing the 29.62 °C early-April 2012 temperature peak of the BWWTP physico-chemical data). Sanitizing is the second of MOVIS's five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing) and is a prerequisite: the analysis core presupposes preprocessed, quality-controlled data. Once sanitizing passes, proceed to the filtering step to restrict to a time-series period (e.g., the beginning of April 2012) and remove unneeded rows or columns.

## Related tools

- **MOVIS** (Web-based, modular time-series multi-omics exploration tool whose sanitizing core step performs the data quality checks) — https://github.com/AAnzel/MOVIS
- **Python** (Implementation language of the MOVIS code (written in Python 3.8.11) that executes the sanitizing logic) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Open-source framework (v1.5.1) underlying MOVIS's interactive web interface through which the sanitizing step is invoked)
- **Pandas** (Library (v1.3.5) providing the Data Frame structures into which tabular CSV/TSV data sets are imported prior to sanitizing)

## Examples

```
docker pull aanzel/movis:latest; docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- Sanitizing confirms the required temporal information feature is present in the tabular data set before the filtering step is attempted
- File names either adhere to MOVIS's naming rules or are automatically corrected during sanitizing
- Downstream filtering of a time-series period (e.g., around the beginning of April 2012) succeeds only after sanitizing has passed
- The subsequent 'Feature through time' visualization of the temperature series reproduces the published trend, peaking at 29.62 °C in early April 2012 (Fig. 8a)
- The exported lossless PNG figure matches the paper-reported case-study visualization on expert review

## Limitations

- Raw microarray-based data cannot be used directly as input; data must already be preprocessed (quality controlled and filtered) before the analysis core
- MOVIS is not explicitly designed to work with meta-omics data — no taxa linking is currently enabled for metagenomics data
- Further omics-wide, time-series-specific, and data-specific standards/guidelines are still required

## Evidence

- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [methods] checking the validity of the file names, and correcting them if they do not adhere to the naming rules: "checking the validity of the file names, and correcting them if they do not adhere to the naming rules"
- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [discussion] the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered.: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] The interactive graphical web interface is built on the open-source framework Streamlit: "The code is written in Python 3.8.11 and tested on Linux with the following libraries installed:"
