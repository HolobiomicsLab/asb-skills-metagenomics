---
name: tabular-data-quality-sanitization
description: Use when a tabular time-series omics dataset (CSV, TSV, or pickled Pandas
  DataFrame with the temporal dimension as a feature column) has just been imported
  via MOVIS's importing step and must be quality-checked before the filtering or analyzing
  steps run.
when_to_use_negative:
- The input is raw microarray-based data, which cannot be used directly as input to
  MOVIS and must be preprocessed externally first
- The data are already preprocessed, quality controlled, and filtered, and the goal
  is direct visualization or clustering (the analyzing/visualizing steps)
- The task is integrative linking across omic types (e.g. taxa linking for metagenomics
  data), which MOVIS does not currently support
edam_operation: http://edamontology.org/operation_2428
tools:
- name: MOVIS
  role: Hosts the sanitizing step as the second of its five sequential core steps
    applied to all imported data sets
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas 1.3.5
  role: Represents the tabular CSV/TSV data sets as Data Frames on which the data-quality
    checks are performed
- name: Streamlit 1.5.1
  role: Web interface through which the import and sanitizing steps are invoked and
    their results presented
- name: Python 3.8.11
  role: Implementation language of the MOVIS core, including the sanitizing routines
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:3335879ad0756a2677a8331adb09a8b9804413bfaa47834112ee59bd743e5ce0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Altair
  - MOVIS
  - Pandas
  - Pandas 1.3.5
  - Python
  - Python 3.8.11
  - Streamlit
  - Streamlit 1.5.1
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# tabular-data-quality-sanitization

## Summary

The sanitizing step of MOVIS's fixed five-step data analysis core (importing, sanitizing, filtering, analyzing, visualizing): it runs data-quality checks on tabular CSV/TSV or pickled DataFrame inputs and validates/corrects file names before any filtering, analysis, or visualization is attempted.

## When to use

Use when a tabular time-series omics dataset (CSV, TSV, or pickled Pandas DataFrame with the temporal dimension as a feature column) has just been imported via MOVIS's importing step and must be quality-checked before the filtering or analyzing steps run.

## When NOT to use

- The input is raw microarray-based data, which cannot be used directly as input to MOVIS and must be preprocessed externally first
- The data are already preprocessed, quality controlled, and filtered, and the goal is direct visualization or clustering (the analyzing/visualizing steps)
- The task is integrative linking across omic types (e.g. taxa linking for metagenomics data), which MOVIS does not currently support

## Inputs

- Tabular CSV or TSV time-series data set, internally imported as a Pandas DataFrame (e.g. the 34 physico-chemical properties of Example 1 with 2-h sampling rate)
- Pickle-serialized Pandas DataFrame from Data/cached/example_1/
- Archived FASTA files (genomics/proteomics) whose file names are checked and corrected during sanitizing

## Outputs

- A quality-checked Pandas DataFrame whose file names adhere to MOVIS naming rules and whose temporal dimension is confirmed present as a feature
- Sanitizing-step error/status feedback that gates entry into the filtering and analyzing steps

## How to apply

Import the tabular dataset with Pandas (MOVIS internally represents CSV/TSV tabular data sets as Pandas Data Frame structures), e.g. the physico-chemical data from the built-in Example 1 (Herold et al. BWWTP multi-meta-omics data, stored under Data/cached/example_1 in raw or pickle format). Then run the sanitizing step, which consists of (1) various data quality checks on the DataFrame and (2) checking the validity of file names and correcting them if they do not adhere to MOVIS's naming rules. Confirm as part of sanitizing that the temporal dimension is present as a feature column of the table, since the downstream filtering step (restricting a time-series period, removing rows/columns) and the analyzing step (KMeans/OPTICS clustering, PCA/MDS/t-SNE projection) presuppose quality-controlled and preprocessed data. Sanitizing applies to tabular data sets; for archived FASTA genomics/proteomics inputs (e.g. Metaproteomics Raw FASTA files), file-name checking is the dominant sanitizing action. Only after sanitizing completes without error should the data proceed to filtering and the visualization step rendered with Altair/Vega-Lite.

## Related tools

- **MOVIS** (Hosts the sanitizing step as the second of its five sequential core steps applied to all imported data sets) — https://github.com/AAnzel/MOVIS
- **Pandas 1.3.5** (Represents the tabular CSV/TSV data sets as Data Frames on which the data-quality checks are performed)
- **Streamlit 1.5.1** (Web interface through which the import and sanitizing steps are invoked and their results presented)
- **Python 3.8.11** (Implementation language of the MOVIS core, including the sanitizing routines)

## Examples

```
import pandas as pd; df = pd.read_csv("example_1_physicochemical.tsv", sep="\t")  # then run MOVIS sanitizing: data quality + file-name checks
```

## Evaluation signals

- All five core steps (importing, sanitizing, filtering, analyzing, visualizing) complete without error on the tabular data set
- Sanitized file names adhere to MOVIS's naming rules (invalid names detected and corrected during sanitizing)
- The temporal dimension is confirmed present as a feature column of the imported DataFrame
- The quality-controlled DataFrame proceeds to the filtering step, which is available only for tabular data sets
- Repeated sanitizing runs over the same cached input (raw or pickle from Data/cached/example_1) are reproducible

## Limitations

- The data analysis core presupposes that the data is preprocessed, i.e. quality controlled and filtered, before use; sanitizing does not replace upstream preprocessing such as for raw microarray-based data
- Sanitizing's file-name checking is defined against MOVIS's naming rules, so data sets violating those rules require correction before or during sanitizing
- Sanitizing does not address integrative multi-omics linking (e.g. no taxa linking for metagenomics data)

## Evidence

- [methods] the sanitizing step consists of various data quality checks: "the sanitizing step consists of various data quality checks"
- [methods] file-name validity checking: "checking the validity of the file names, and correcting them if they do not adhere to the naming rules"
- [methods] Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [discussion] preprocessing precondition: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [readme] Example 1 data location and formats: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
