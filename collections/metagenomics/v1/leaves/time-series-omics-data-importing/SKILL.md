---
name: time-series-omics-data-importing
description: Use when a tabular time-series omics data set (CSV/TSV or pickled Pandas
  DataFrame) or archived FASTA files must be brought into a Pandas DataFrame pipeline
  as the first of MOVIS's five sequential core steps.
when_to_use_negative:
- Input is raw microarray-based data, which cannot be used directly as input to MOVIS
- Data lacks an embedded temporal dimension, so no time-series importing/analysis
  is possible
- The task requires direct integrative linking of multi-omics data (e.g., taxa linking
  for metagenomics), which is planned but not yet available in MOVIS
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Provides the importing step of the fixed five-step data analysis core and
    the upload page for user-supplied data sets
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Internally represents imported tabular CSV/TSV data sets as Pandas Data Frame
    structures
- name: Streamlit
  role: Web interface through which users upload data sets to the importing step
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:60398b7f8ad2d50fd5c23fdfbfba06d83deea6df566e233bdd654c65f087232e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# time-series-omics-data-importing

## Summary

The importing step of MOVIS's fixed five-step data analysis core: it loads time-series multi-omics inputs — archived FASTA files for genomics/proteomics and tabular CSV/TSV files (internally as Pandas Data Frames) for other omic-types — as the entry point to importing, sanitizing, filtering, analyzing, and visualizing.

## When to use

Use when a tabular time-series omics data set (CSV/TSV or pickled Pandas DataFrame) or archived FASTA files must be brought into a Pandas DataFrame pipeline as the first of MOVIS's five sequential core steps.

## When NOT to use

- Input is raw microarray-based data, which cannot be used directly as input to MOVIS
- Data lacks an embedded temporal dimension, so no time-series importing/analysis is possible
- The task requires direct integrative linking of multi-omics data (e.g., taxa linking for metagenomics), which is planned but not yet available in MOVIS

## Inputs

- Archived FASTA files (genomics and proteomics data sets)
- Tabular CSV or TSV files with an embedded temporal dimension as a feature
- Pickled Pandas DataFrame objects from Data/cached/example_1 or Data/cached/example_2
- Time-series multi-omics data sets from Herold et al. 2020 (BWWTP) and Merchel Piovesan Pereira et al. 2020 (E. coli biocide response)

## Outputs

- Pandas Data Frame structures for each imported tabular data set
- Unpacked FASTA-derived tabular data sets (e.g., Word2Vec embeddings with one row per FASTA file and 101 columns)
- Imported data sets passed to the sanitizing step for data quality and file-name checks

## How to apply

Invoke importing as the first of MOVIS's five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing), which are fixed for all data sets. For tabular omic types (e.g., the physico-chemical or metabolomics data in Data/cached/example_1, derived from Herold et al.'s BWWTP meta-omics data set), read CSV or TSV files into Pandas Data Frame structures, or load the pre-cached pickle objects shipped with the repository. For genomics and proteomics inputs, provide archived FASTA files, which MOVIS unpacks and converts — e.g., via the Word2Vec embedding — into tabular data sets with one row per FASTA file. Ensure the data carries an embedded temporal dimension, since downstream steps (filtering by time-series period, time-series visualization) presuppose it; note that the core presupposes data is quality controlled and filtered, which the subsequent sanitizing and filtering steps perform. Data sets must follow the cached layout (e.g., Data/cached/example_1) or be supplied through the Streamlit upload page so the fixed pipeline can proceed reproducibly.

## Related tools

- **MOVIS** (Provides the importing step of the fixed five-step data analysis core and the upload page for user-supplied data sets) — https://github.com/AAnzel/MOVIS
- **Pandas** (Internally represents imported tabular CSV/TSV data sets as Pandas Data Frame structures)
- **Streamlit** (Web interface through which users upload data sets to the importing step)

## Examples

```
import pandas as pd; df = pd.read_csv('Data/cached/example_1/physico-chemical.tsv', sep='\t')  # MOVIS importing: tabular CSV/TSV -> Pandas DataFrame
```

## Evaluation signals

- Each CSV/TSV file loads without error into a Pandas DataFrame with the temporal dimension present as a feature column
- Archived FASTA input is unpacked and converted to a tabular data set with the same number of rows as FASTA files in the archived data set
- Imported data passes the subsequent sanitizing step's data quality checks and file-name validity checks without errors
- Loading Data/cached/example_1 (raw or pickle form) yields a data set equivalent to the published processed data sets (e.g., 34 physico-chemical properties with 2-h sampling rate)
- The run proceeds through all five sequential core steps (importing, sanitizing, filtering, analyzing, visualizing) without error

## Limitations

- Raw microarray-based data cannot be used directly as input; preprocessing is required
- The data analysis core presupposes that the data is preprocessed, i.e., quality controlled and filtered
- Only two time-series multi-omics data sets have been integrated and reproduced so far
- No taxa linking is enabled for metagenomics data, and direct multi-omics integration is not yet available

## Evidence

- [methods] Five-step core definition: [evidence span withheld — anchor status: not_found]
- [methods] Tabular import format: "Tabular data sets (CSV or TSV) are internally imported as Pandas Data Frame structures"
- [methods] FASTA import: "In the first core step, i.e., importing, genomics, and proteomics data sets can be provided as archived FASTA files"
- [readme] Caching of Example 1 data: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
- [discussion] Preprocessing presupposition: "the data analysis core presupposes that the data is preprocessed, that is to say, quality controlled and filtered."
