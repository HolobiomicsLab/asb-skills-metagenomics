---
name: temporal-feature-quality-checking
description: Use when importing a raw or pickled CSV/TSV time-series multi-omics table
  into MOVIS and you must confirm data quality and that the temporal feature is detected
  before filtering or plotting.
when_to_use_negative:
- Input is raw microarray-based data, which cannot be used directly as input per the
  MOVIS discussion section
- Input has not been preprocessed at all — the core presupposes quality controlled
  and filtered data, so run external preprocessing first
- Goal is taxa linking for metagenomics data — MOVIS is not explicitly designed for
  this and no taxa linking is currently enabled
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3077
tools:
- name: MOVIS
  role: Hosts the importing and sanitizing core steps that run data quality checks
    and confirm the embedded temporal feature of tabular time-series data
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Internally imports the CSV/TSV tabular datasets as Pandas DataFrame structures
    on which quality checks operate (v1.3.5)
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Provides the interactive web interface through which the sanitizing and filtering
    steps are executed (v1.5.1)
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:e7674431558074e3f34ae215f174057d453b22d52a86842e9519b5dbe4bde19a
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
yes_rate: 0.45454545454545453
n_claims: 11
---

# temporal-feature-quality-checking

## Summary

Validates that a tabular time-series multi-omics dataset (e.g., MOVIS Example 1 metagenomics depth-of-coverage data from Data/cached/example_1/) passes the MOVIS sanitizing data quality checks and that its embedded temporal feature is correctly recognized before any visualization or clustering step.

## When to use

Use when importing a raw or pickled CSV/TSV time-series multi-omics table into MOVIS and you must confirm data quality and that the temporal feature is detected before filtering or plotting.

## When NOT to use

- Input is raw microarray-based data, which cannot be used directly as input per the MOVIS discussion section
- Input has not been preprocessed at all — the core presupposes quality controlled and filtered data, so run external preprocessing first
- Goal is taxa linking for metagenomics data — MOVIS is not explicitly designed for this and no taxa linking is currently enabled

## Inputs

- Raw or pickled tabular (CSV/TSV) time-series dataset from Data/cached/example_1/ (e.g., Metagenomics Depth-of-coverage data)
- FASTA-derived physico-chemical feature tables (e.g., 43 proteomics features) when checking embedded temporal structure

## Outputs

- Quality-checked Pandas DataFrame with a confirmed embedded temporal feature
- Sanitized/validated dataset ready for MOVIS filtering, analysis, and visualization steps

## How to apply

Load the Example 1 (BWWTP) dataset — e.g., the metagenomics Depth-of-coverage tables stored raw or as pickle objects under Data/cached/example_1/ — and import it into the MOVIS data analysis core (Streamlit 1.5.1 + Pandas 1.3.5). Run the MOVIS sanitizing step, which consists of various data quality checks, including checking the validity of file names and correcting names that do not adhere to naming rules; this step must confirm the embedded temporal feature of the tabular data. Note that the data analysis core presupposes the data is already preprocessed (quality controlled and filtered), so raw microarray-type inputs are not acceptable directly. If needed, proceed to the tabular-only filtering step to restrict a time-series period or drop rows/columns, then re-run sanitization on the derived dataset. Only after quality confirmation should the series be passed to analysis/visualization (e.g., Whisker plot) to expose temporal patterns such as the ~one-month sine-curve in Q3 and upper limits.

## Related tools

- **MOVIS** (Hosts the importing and sanitizing core steps that run data quality checks and confirm the embedded temporal feature of tabular time-series data) — https://github.com/AAnzel/MOVIS
- **Pandas** (Internally imports the CSV/TSV tabular datasets as Pandas DataFrame structures on which quality checks operate (v1.3.5)) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Provides the interactive web interface through which the sanitizing and filtering steps are executed (v1.5.1)) — https://github.com/AAnzel/MOVIS

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- Sanitizing step completes with no naming-rule violations, or file names are automatically corrected to adhere to MOVIS naming rules
- The embedded temporal feature of the tabular dataset is confirmed/detected during sanitization before proceeding to filtering
- Subsequent time-series visualization (e.g., Whisker plot of metagenomics depth-of-coverage) reproduces the reported Q3/upper-limit sine-curve with roughly one-month period
- Any derived dataset created via filtering re-passes quality checks before analysis

## Limitations

- The data analysis core presupposes that data is preprocessed (quality controlled and filtered) before use
- Specific data such as raw microarray-based data cannot be used directly as input
- MOVIS does not currently support taxa linking for metagenomics data, so quality checking cannot validate taxonomic consistency

## Evidence

- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [methods] Sanitizing quality checks include file-name validity checking: "checking the validity of the file names, and correcting them if they do not adhere to the naming rules"
- [methods] Filtering is restricted to tabular data sets and supports time-period restriction: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [discussion] Core presupposes preprocessed, quality-controlled data: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] Example 1 data stored raw or as pickle under Data/cached/example_1/: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
