---
name: temporal-trend-detection-in-metabolomics
description: Use when a tabular (CSV/TSV) time-series metabolomics dataset must be
  inspected for temporal substrate shifts or trend discontinuities in specific date
  windows, e.g., the November–December 2011 BWWTP sludge data.
when_to_use_negative:
- 'Data is not preprocessed: the data analysis core presupposes quality-controlled
  and filtered input, so raw microarray-based data or unprocessed FASTA archives should
  not be fed directly into this workflow'
- The analysis goal is direct integrative linking across omics types — MOVIS currently
  enables no taxa linking and direct multi-omics integration is planned but not yet
  available
- The dataset is a non-tabular type for which the filtering step is unavailable (filtering
  is present only for tabular data sets)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Modular web-based tool used to import, sanitize, filter, cluster, and visualize
    the metabolomics time series
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit
  role: Framework underlying the interactive graphical web interface through which
    the workflow steps are executed
- name: Pandas
  role: Internally imports the tabular CSV/TSV metabolomics dataset as Pandas DataFrame
    structures
- name: Scikit-learn
  role: Provides the optional KMeans (distance-based) and OPTICS (density-based) clustering
    of the filtered time series
- name: Altair
  role: Renders the Time heatmap and other visualizations with tooltip/brushing/zooming
    interactivity and PNG export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:ebe89acb19c95d7340a1127bb58adec5df125c8e104454a026875d75163b6a0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Altair
  - MOVIS
  - Pandas
  - Python
  - Scikit-learn
  - Streamlit
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Temporal trend detection in metabolomics

## Summary

Reproduce substrate-shift detection in the BWWTP metabolomics time series (Processed data set 2) using the MOVIS web tool: import the cached Example 1 tabular dataset, sanitize, filter to the target time window, optionally cluster, and render the Time heatmap to reveal temporal discontinuities.

## When to use

Use when a tabular (CSV/TSV) time-series metabolomics dataset must be inspected for temporal substrate shifts or trend discontinuities in specific date windows, e.g., the November–December 2011 BWWTP sludge data.

## When NOT to use

- Data is not preprocessed: the data analysis core presupposes quality-controlled and filtered input, so raw microarray-based data or unprocessed FASTA archives should not be fed directly into this workflow
- The analysis goal is direct integrative linking across omics types — MOVIS currently enables no taxa linking and direct multi-omics integration is planned but not yet available
- The dataset is a non-tabular type for which the filtering step is unavailable (filtering is present only for tabular data sets)

## Inputs

- Tabular metabolomics time-series dataset (CSV or TSV) from Data/cached/example_1 (BWWTP Processed data set 2, ~95% metabolomics)
- Target time window definition (early-to-mid November 2011 and early-to-mid December 2011)

## Outputs

- MOVIS Time heatmap visualization of the metabolomics time series with visible substrate-shift windows
- Exported publication-ready PNG figure
- Optionally, cluster assignments from KMeans or OPTICS applied to the filtered series

## How to apply

Launch MOVIS (Streamlit interface, e.g., via the Docker image or the online instance at movis.mathematik.uni-marburg.de) and import the Example 1 metabolomics dataset from Data/cached/example_1 as a tabular CSV/TSV file, which is internally loaded as a Pandas DataFrame. Run the sanitizing step, which performs data quality checks including file name validity checking and correction. Apply the filtering step (available only for tabular data) to restrict the time-series period to the November–December 2011 window and remove unneeded rows or columns. Optionally apply the analysis core's clustering (KMeans, distance-based, or OPTICS, density-based) via scikit-learn to the prepared series. Render the Time heatmap on the visualization canvas (Altair-based, with tooltip/brushing/zooming) and export the figure as PNG, then confirm the reported shift signal appears.

## Related tools

- **MOVIS** (Modular web-based tool used to import, sanitize, filter, cluster, and visualize the metabolomics time series) — https://github.com/AAnzel/MOVIS
- **Streamlit** (Framework underlying the interactive graphical web interface through which the workflow steps are executed)
- **Pandas** (Internally imports the tabular CSV/TSV metabolomics dataset as Pandas DataFrame structures)
- **Scikit-learn** (Provides the optional KMeans (distance-based) and OPTICS (density-based) clustering of the filtered time series)
- **Altair** (Renders the Time heatmap and other visualizations with tooltip/brushing/zooming interactivity and PNG export)

## Examples

```
docker pull aanzel/movis:latest; docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest
```

## Evaluation signals

- The exported Time heatmap reproduces the reported substrate-shift signal: a visible temporal trend discontinuity between early and mid-November 2011 and between early and mid-December 2011, with noticeably higher values in between
- Values normalize after the end of December 2011, matching the reported finding
- The sanitizing step completes without unresolvable data-quality or file-name validity errors on the imported dataset
- The filtered dataset is restricted to the November–December 2011 window with unneeded rows/columns removed before visualization
- The exported figure is publication-ready PNG output consistent with the paper's reproduced visualization of Processed data set 2

## Limitations

- MOVIS presupposes preprocessed input: the data analysis core requires quality-controlled, filtered data, and raw microarray-based data cannot be used directly
- No taxa linking is enabled for metagenomics data, and direct integrative multi-omics analysis is planned but not yet available
- The filtering step exists only for tabular data sets, so non-tabular inputs cannot be period-filtered in this way
- The finding is based on visual inspection of the Time heatmap; two example datasets have been integrated and reproduced, but further data-specific standards are required per the discussion

## Evidence

- [results] Inspection of the MOVIS Time heatmap visualization of Processed data set 2 revealed substrate shifts from early to mid-November 2011 and from early to mid-December 2011: [evidence span withheld — anchor status: not_found]
- [methods] The filtering step is present only for tabular data sets: "The filtering step is present only for tabular data sets"
- [methods] It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [methods] The core works with five sequential core steps: importing, sanitizing, filtering, analyzing, and visualizing all data sets: [evidence span withheld — anchor status: not_found]
- [results] Almost 95% of the data set represents metabolomics data: "Almost 95% of the data set represents metabolomics data"
- [readme] README dependency table listing streamlit 1.5.1 and pandas 1.3.5: "The interactive graphical web interface is built on the open-source framework Streamlit"
