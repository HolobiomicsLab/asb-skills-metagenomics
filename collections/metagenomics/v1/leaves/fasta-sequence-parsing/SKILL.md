---
name: fasta-sequence-parsing
description: Use when the input is archived raw FASTA files (genomics or proteomics
  omic-type) that must be parsed into individual sequences before embedding, vectorization,
  or tabular conversion in a time-series multi-omics pipeline.
when_to_use_negative:
- The input is already a tabular data set (CSV/TSV feature table) — the filtering
  step applies only to tabular data and FASTA parsing is unnecessary.
- The data are raw microarray-based data, which cannot be used directly as input and
  require separate preprocessing before the analysis core.
- The data have not been quality controlled and filtered — the data analysis core
  presupposes preprocessed data.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Biopython
  role: Parses each FASTA sequence in the archived data set using SeqIO as the first
    analysis-core step before tokenization and embedding
  repo: https://github.com/AAnzel/MOVIS
- name: Gensim
  role: Trains a Word2Vec model on the k-mer token corpus and embeds each parsed sequence
    as a 100-dimensional vector
  repo: https://github.com/AAnzel/MOVIS
- name: Pandas
  role: Assembles the per-file averaged embedding vectors into a 101-column DataFrame
    saved as embeddings.csv
- name: MOVIS
  role: Provides the importing/sanitizing conventions for archived FASTA files and
    hosts the data analysis core in which this parsing occurs
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:4272230fa9856a317cad351695508e21ca0bd830a48211534192e1772497e7f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AAnzel/MOVIS
  tools:
  - Biopython
  - Gensim
  - MOVIS
  - NumPy
  - Pandas
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fasta-sequence-parsing

## Summary

Parse archived FASTA sequence files (e.g., the Example 1 BWWTP metaproteomics raw FASTA data in MOVIS) with Biopython's SeqIO to extract individual sequences, which are then tokenized and embedded (e.g., via Gensim Word2Vec into 100-dimensional vectors) for downstream clustering.

## When to use

Use when the input is archived raw FASTA files (genomics or proteomics omic-type) that must be parsed into individual sequences before embedding, vectorization, or tabular conversion in a time-series multi-omics pipeline.

## When NOT to use

- The input is already a tabular data set (CSV/TSV feature table) — the filtering step applies only to tabular data and FASTA parsing is unnecessary.
- The data are raw microarray-based data, which cannot be used directly as input and require separate preprocessing before the analysis core.
- The data have not been quality controlled and filtered — the data analysis core presupposes preprocessed data.

## Inputs

- Archived raw FASTA files (zip/tar archives of FASTA sequence files, e.g., Example 1 BWWTP metaproteomics raw FASTA data)
- Temporal metadata associated with each FASTA file

## Outputs

- Parsed sequence records per FASTA file (Biopython SeqIO objects)
- Tokenized k-mer corpus per sequence
- 100-dimensional Word2Vec embedding per sequence
- One averaged 100-dimensional vector per FASTA file
- embeddings.csv: Pandas DataFrame with one row per FASTA file and 101 columns (1 temporal + 100 embedding dimensions)

## How to apply

Unpack the archived FASTA files (e.g., from Data/cached/example_1/) with Python's built-in zipfile/tarfile libraries, following MOVIS's sanitizing conventions for archived data. Parse each contained sequence using Biopython's SeqIO, then tokenize each nucleotide/amino-acid sequence into overlapping k-mer tokens to form a token corpus. Train a Gensim Word2Vec model on this corpus and use it to embed each parsed sequence as a 100-dimensional vector (the dimension chosen empirically in the paper). Average per-sequence vectors within each FASTA file to obtain one representative vector per file, assembling a Pandas DataFrame with one row per FASTA file and 101 columns (one temporal column plus 100 embedding dimensions) written to embeddings.csv.

## Related tools

- **Biopython** (Parses each FASTA sequence in the archived data set using SeqIO as the first analysis-core step before tokenization and embedding) — https://github.com/AAnzel/MOVIS
- **Gensim** (Trains a Word2Vec model on the k-mer token corpus and embeds each parsed sequence as a 100-dimensional vector) — https://github.com/AAnzel/MOVIS
- **Pandas** (Assembles the per-file averaged embedding vectors into a 101-column DataFrame saved as embeddings.csv)
- **MOVIS** (Provides the importing/sanitizing conventions for archived FASTA files and hosts the data analysis core in which this parsing occurs) — https://github.com/AAnzel/MOVIS

## Examples

```
from Bio import SeqIO; records = [r for f in fasta_files for r in SeqIO.parse(f, 'fasta')]
```

## Evaluation signals

- embeddings.csv exists and has exactly 101 columns (one temporal column plus 100 embedding-dimension columns).
- The row count of embeddings.csv equals the number of FASTA files in the input archive.
- Every parsed FASTA file yields exactly one averaged 100-dimensional representative vector.
- Downstream clustering on the parsed-and-embedded vectors reproduces the reference silhouette score of 0.495 for Example 1 metaproteomics raw FASTA with KMeans k=3 within a small numerical tolerance.
- Archives are unpacked and sanitized according to MOVIS conventions (file name validity checks and correction).

## Limitations

- Word2Vec embedding of 100 dimensions was chosen empirically per the paper, not derived from first principles.
- MOVIS is not explicitly designed to work with meta-omics data, so no taxa linking is enabled for metagenomics FASTA inputs.
- The data analysis core presupposes preprocessed data — quality controlled and filtered; raw, unchecked FASTA archives may fail sanitization.
- Raw microarray-based and similar specific data formats cannot be used directly as input.

## Evidence

- [methods] In the first core step, i.e., importing, genomics, and proteomics data sets can be provided as archived FASTA files: "In the first core step, i.e., importing, genomics, and proteomics data sets can be provided as archived FASTA files"
- [methods] Word2Vec tabular output shape: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [results] Metaproteomics raw FASTA selection in Example 1: "we selected Metaproteomics data, and then Raw FASTA files."
- [methods] Sanitizing quality checks: "the sanitizing step consists of various data quality checks"
- [readme] Example 1 data location in repository: "It is stored at [Data/cached/example_1/](./Data/cached/example_1) in either a raw format or as a [pickle](https://docs.python.org/3/library/pickle.html) object."
