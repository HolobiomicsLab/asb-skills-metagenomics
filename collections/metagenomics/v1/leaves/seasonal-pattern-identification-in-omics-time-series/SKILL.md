---
name: seasonal-pattern-identification-in-omics-time-series
description: Use when a tabular time-series omics dataset with an embedded temporal
  feature (e.g., Herold et al. BWWTP metagenomics depth-of-coverage) should be screened
  for periodic or seasonal dynamics such as quartile envelopes tracing a sine-like
  curve.
when_to_use_negative:
- The input data has no embedded temporal feature — MOVIS's sanitizing step presupposes
  time-series data that passes its data quality checks
- Raw microarray-based data is the input, since such data cannot be used directly
  as input to MOVIS and must be preprocessed (quality controlled and filtered)
- The goal is linking taxa in metagenomics data — MOVIS is not explicitly designed
  to work with meta-omics data and no taxa linking is enabled
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Web-based, modular time-series multi-omics exploration tool whose importing/sanitizing/filtering/analyzing/visualizing
    core renders the Whisker plot exposing the sine-curve pattern
  repo: https://github.com/AAnzel/MOVIS
- name: Streamlit 1.5.1
  role: Interactive graphical web interface framework underlying the MOVIS visualization
    canvas used to inspect the quartile envelopes
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas 1.3.5
  role: Internally imports the CSV/TSV depth-of-coverage dataset as Pandas DataFrame
    structures inside the MOVIS core
- name: Altair 4.1.0
  role: Plotting library (via Source/visualize.py) that creates the Whisker plot visualization
    of the depth-of-coverage quartiles
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:672dfea1e8b9373f8fb0826734f9a567cd3c2ab9d25cda197d10d787d42a26c2
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# seasonal-pattern-identification-in-omics-time-series

## Summary

Identify recurring (e.g., ~monthly sine-curve) temporal patterns in time-series multi-omics data by importing the tabular depth-of-coverage series into MOVIS, passing it through the importing/sanitizing/filtering core, and inspecting quartile envelopes (Q3 and upper limits) in a Whisker plot visualization.

## When to use

Use when a tabular time-series omics dataset with an embedded temporal feature (e.g., Herold et al. BWWTP metagenomics depth-of-coverage) should be screened for periodic or seasonal dynamics such as quartile envelopes tracing a sine-like curve.

## When NOT to use

- The input data has no embedded temporal feature — MOVIS's sanitizing step presupposes time-series data that passes its data quality checks
- Raw microarray-based data is the input, since such data cannot be used directly as input to MOVIS and must be preprocessed (quality controlled and filtered)
- The goal is linking taxa in metagenomics data — MOVIS is not explicitly designed to work with meta-omics data and no taxa linking is enabled

## Inputs

- Tabular time-series metagenomics depth-of-coverage dataset (CSV/TSV or pickled DataFrame) from the BWWTP multi-meta-omics study (Herold et al. 2020), stored under Data/cached/example_1/

## Outputs

- MOVIS Whisker plot (or Feature through time) figure showing Q3 and upper-limit quartile envelopes as a function of time, exported as a PNG
- Qualitative identification of a periodic (roughly one-month period) sine-curve pattern in the population-dynamics quartiles

## How to apply

Load the tabular dataset from Data/cached/example_1/ (raw or pickled form) and import it into the MOVIS data analysis core, which is built on Streamlit 1.5.1 and Pandas 1.3.5 and works with five sequential steps: importing, sanitizing, filtering, analyzing, and visualizing. The sanitizing step runs data quality checks and confirms the embedded temporal feature; the optional tabular-only filtering step can restrict the time-series period or remove rows/columns. Select a time-series visualization from MOVIS's nine supported plot types — the Whisker plot exposes quartile envelopes of the depth-of-coverage series, so inspect the third quartile and upper limit traces for sinusoidal behavior. Export the resulting figure via the visualization canvas as a publication-ready PNG and compare it against the reported MOVIS visualization.

## Related tools

- **MOVIS** (Web-based, modular time-series multi-omics exploration tool whose importing/sanitizing/filtering/analyzing/visualizing core renders the Whisker plot exposing the sine-curve pattern) — https://github.com/AAnzel/MOVIS
- **Streamlit 1.5.1** (Interactive graphical web interface framework underlying the MOVIS visualization canvas used to inspect the quartile envelopes) — https://github.com/AAnzel/MOVIS
- **Pandas 1.3.5** (Internally imports the CSV/TSV depth-of-coverage dataset as Pandas DataFrame structures inside the MOVIS core)
- **Altair 4.1.0** (Plotting library (via Source/visualize.py) that creates the Whisker plot visualization of the depth-of-coverage quartiles) — https://github.com/AAnzel/MOVIS

## Examples

```
docker pull aanzel/movis:latest && docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest  # then open http://localhost:8501/, load Data/cached/example_1 metagenomics Depth-of-coverage, plot Whisker
```

## Evaluation signals

- The exported PNG reproduces the reported MOVIS Example 1 visualization: Q3 and upper-limit traces form a sine-curve-like pattern with a period of roughly one month
- A slight discrepancy appears around the beginning of November 2011, attributable to increased outliers — its presence confirms faithful reproduction
- The dataset passes MOVIS's sanitizing step (data quality checks confirm the embedded temporal feature) before visualization
- The Whisker plot's quartile envelopes are computed from the full depth-of-coverage series, so filtering that restricts the time-series period should visibly shrink the plotted time range without breaking the periodic shape within the retained window

## Limitations

- The pattern read-out is visual (Whisker quartile envelopes), not a formal statistical periodicity test, so confidence rests on reproducing the published figure
- Outlier density can distort the apparent phase of the Q3/upper-limit curve (as around the beginning of November 2011 in Example 1)
- The filtering step exists only for tabular data sets, so this workflow cannot be applied to non-tabular FASTA-derived inputs without preprocessing

## Evidence

- [methods] sequential core steps: [evidence span withheld — anchor status: not_found]
- [results] depth-of-coverage selection: "we selected Metagenomics data, and then Depth-of-coverage, which presented us with a directory hierarchy of the underlying data set."
- [methods] whisker plot period: "It offers a way to filter a certain time-series period and remove one or more rows or columns from the data set"
- [readme] example_1 data location: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
- [readme] docker run: "docker pull aanzel/movis:latest
docker run --publish 8501:8501 --detach --name movis aanzel/movis:latest"
