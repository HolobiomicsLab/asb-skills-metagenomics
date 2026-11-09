---
name: time-series-period-filtering
description: Use when a tabular CSV/TSV time-series dataset (e.g., the BWWTP physico-chemical
  data with 2-h sampling) contains time ranges or features/rows outside the analytical
  scope and must be subset before clustering or visualization.
when_to_use_negative:
- Input is archived genomics/proteomics FASTA data — the filtering step is present
  only for tabular data sets
- Raw microarray-based data, which cannot be used directly as input and requires external
  preprocessing before MOVIS
- Data that has not been quality controlled — the core presupposes preprocessed (quality
  controlled and filtered) input
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: MOVIS
  role: Web-based modular tool whose filtering step restricts the time-series period
    and removes rows/columns from tabular DataFrames
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas 1.3.5
  role: Underlying DataFrame structures on which importing and row/column removal
    operate
- name: Streamlit 1.5.1
  role: Interactive web interface through which the filtering step is invoked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:95e85b2da5673a7309497a3ed2fe12cd88895894a91900b7bb20acc0f9419c77
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
  - Streamlit
  - Streamlit 1.5.1
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# time-series-period-filtering

## Summary

MOVIS's filtering step restricts a Pandas DataFrame time-series dataset to a chosen temporal period and optionally removes selected rows or columns, applied between sanitizing and analyzing in the five-step core (importing, sanitizing, filtering, analyzing, visualizing).

## When to use

Use when a tabular CSV/TSV time-series dataset (e.g., the BWWTP physico-chemical data with 2-h sampling) contains time ranges or features/rows outside the analytical scope and must be subset before clustering or visualization.

## When NOT to use

- Input is archived genomics/proteomics FASTA data — the filtering step is present only for tabular data sets
- Raw microarray-based data, which cannot be used directly as input and requires external preprocessing before MOVIS
- Data that has not been quality controlled — the core presupposes preprocessed (quality controlled and filtered) input

## Inputs

- Tabular CSV or TSV time-series dataset internally imported as a Pandas DataFrame (e.g., Data/cached/example_1 physico-chemical data with 34 properties at 2-h sampling rate)
- Sanitized DataFrame that has passed data quality and file-name checks with a temporal feature confirmed

## Outputs

- Filtered Pandas DataFrame restricted to the chosen time-series period with optionally removed rows or columns
- Filtered dataset forwarded to the analyzing step (optional KMeans/OPTICS clustering, PCA/MDS/t-SNE projection) and then the visualization step

## How to apply

Load the tabular dataset as a Pandas DataFrame via MOVIS's importing step (CSV or TSV), then pass it through the sanitizing step (data quality checks, file-name validity checking, and confirmation of the temporal feature). In the filtering step, restrict the DataFrame to a chosen time-series period and optionally drop one or more rows or columns; note that filtering is available only for tabular data sets, not for FASTA-derived or raw FASTA inputs. Rationale: MOVIS presupposes quality-controlled, preprocessed data before the analyzing step (optional KMeans/OPTICS clustering and PCA/MDS/t-SNE projection), so period filtering ensures only the temporal window of interest enters the visualization step rendered with Altair/Vega-Lite.

## Related tools

- **MOVIS** (Web-based modular tool whose filtering step restricts the time-series period and removes rows/columns from tabular DataFrames) — https://github.com/AAnzel/MOVIS
- **Pandas 1.3.5** (Underlying DataFrame structures on which importing and row/column removal operate)
- **Streamlit 1.5.1** (Interactive web interface through which the filtering step is invoked)

## Examples

```
docker pull aanzel/movis:latest; docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- Filtered DataFrame contains only rows within the chosen time-series period and excludes the removed rows/columns
- The temporal dimension remains present as a feature after filtering
- Downstream analyzing step (KMeans/OPTICS clustering, PCA/MDS/t-SNE) runs without error on the filtered data
- Rendered Altair/Vega-Lite visualization matches the filtered temporal window and remains exportable as PNG (or SVG/Vega-Lite spec)
- All five core steps (importing, sanitizing, filtering, analyzing, visualizing) complete without error

## Limitations

- Filtering is available only for tabular data sets; FASTA-based datasets cannot be period-filtered inside MOVIS
- The data analysis core presupposes that data is already preprocessed (quality controlled and filtered) before use
- MOVIS does not directly integrate multi-omics data (e.g., no taxa linking for metagenomics); filtering applies per data set

## Evidence

- [methods] filtering is available only for tabular data sets: "The filtering step is present only for tabular data sets"
- [methods] period filtering and row/column removal: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [methods] five sequential core steps: [evidence span withheld — anchor status: not_found]
- [discussion] preprocessing presupposition: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
- [methods] Pandas DataFrame import: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
