---
name: multi-sequence-vector-averaging
description: Use when input is archived (raw) FASTA files from time-series multi-omics
  data (e.g., metaproteomics) and the goal is to convert sequences into a numeric
  feature table suitable for downstream clustering or visualization.
when_to_use_negative:
- The data are already tabular (CSV/TSV feature tables) — MOVIS's filtering step is
  present only for tabular data sets and FASTA embedding is unnecessary.
- The input is raw microarray-based data, which MOVIS cannot use directly as input
  and requires preprocessing first.
- Only per-sequence vectors (not per-file representative vectors) are needed, since
  this skill specifically averages vectors within each FASTA file.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: MOVIS
  role: Modular multi-omics tool whose data analysis core performs the FASTA embedding
    step prior to clustering
  repo: https://github.com/AAnzel/MOVIS
- name: Gensim
  role: Word2Vec model trained on k-mer token corpus; used to embed each sequence
    as a 100-dimensional vector
- name: Biopython
  role: SeqIO parsing of each contained sequence from the archived FASTA files
- name: Pandas
  role: Assembles the 101-column DataFrame and writes it to embeddings.csv
- name: NumPy
  role: Numerical averaging of per-sequence vectors within each FASTA file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:cddc31a7d8518f9e147acc2fe22720f2c813c5b1e9f6816e8214b752b2f0f2fc
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# multi-sequence-vector-averaging

## Summary

Embeds FASTA sequences into 100-dimensional Word2Vec vectors and averages the per-sequence vectors within each FASTA file to yield one representative vector per file, enabling tabular clustering (e.g., K-Means with the Elbow rule) in MOVIS's data analysis core.

## When to use

Use when input is archived (raw) FASTA files from time-series multi-omics data (e.g., metaproteomics) and the goal is to convert sequences into a numeric feature table suitable for downstream clustering or visualization.

## When NOT to use

- The data are already tabular (CSV/TSV feature tables) — MOVIS's filtering step is present only for tabular data sets and FASTA embedding is unnecessary.
- The input is raw microarray-based data, which MOVIS cannot use directly as input and requires preprocessing first.
- Only per-sequence vectors (not per-file representative vectors) are needed, since this skill specifically averages vectors within each FASTA file.

## Inputs

- archived (raw) FASTA files from the time-series multi-omics data set (e.g., Example 1 metaproteomics Raw FASTA files)
- temporal information for each FASTA file (to occupy the time column of the output table)

## Outputs

- embeddings.csv: a Pandas DataFrame tabular data set with one row per FASTA file and 101 columns (1 temporal column + 100 Word2Vec embedding-dimension columns)

## How to apply

Parse each FASTA sequence from the archived FASTA files with Biopython's SeqIO, tokenize each sequence into overlapping k-mer tokens, and train a Gensim Word2Vec model on the resulting token corpus. Use the trained model to embed each parsed sequence as a 100-dimensional vector, then average all per-sequence vectors within each FASTA file so that every file yields exactly one representative vector. Assemble a Pandas DataFrame with one row per FASTA file and 101 columns: one column holding the temporal information plus the 100 embedding-dimension columns, and write it to embeddings.csv. The embedded data can then be passed to a clustering method such as K-Means, with the number of clusters chosen via the Elbow rule chart.

## Related tools

- **MOVIS** (Modular multi-omics tool whose data analysis core performs the FASTA embedding step prior to clustering) — https://github.com/AAnzel/MOVIS
- **Gensim** (Word2Vec model trained on k-mer token corpus; used to embed each sequence as a 100-dimensional vector)
- **Biopython** (SeqIO parsing of each contained sequence from the archived FASTA files)
- **Pandas** (Assembles the 101-column DataFrame and writes it to embeddings.csv)
- **NumPy** (Numerical averaging of per-sequence vectors within each FASTA file)

## Examples

```
from Bio import SeqIO; from gensim.models import Word2Vec; seqs = [str(rec.seq) for rec in SeqIO.parse('example_1.fasta', 'fasta')]; model = Word2Vec([list(s) for s in seqs], vector_size=100)
```

## Evaluation signals

- embeddings.csv exists after the embedding step completes
- embeddings.csv has exactly 101 columns (1 temporal + 100 embedding dimensions)
- Row count of embeddings.csv equals the number of FASTA files in the input archive
- Each row's 100 embedding values correspond to the average of per-sequence Word2Vec vectors within the corresponding FASTA file
- The embedded data can subsequently be clustered with K-Means, with cluster count chosen via the Elbow rule chart

## Limitations

- Data must be preprocessed (quality controlled and filtered) before use; the data analysis core presupposes preprocessed input.
- MOVIS is not explicitly designed to work with meta-omics data, so no taxa linking is currently enabled for metagenomics data.
- Direct integrative aspect of multi-omics data is planned but not yet directly available in MOVIS.
- The embedding produces one representative vector per FASTA file via averaging, so within-file sequence-level variation is collapsed.

## Evidence

- [methods] MOVIS operates on raw FASTA input by embedding the FASTA files into vector representations; once the embedding step completes, the resulting embedded data can be passed to a clustering method such as K-Means, with the number of clusters chosen via the Elbow rule chart.: [evidence span withheld — anchor status: not_found]
- [methods] Word2Vec embedding produces a tabular dataset with one row per FASTA file and 101 columns: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [methods] In the first core step, i.e., importing, genomics, and proteomics data sets can be provided as archived FASTA files: "In the first core step, i.e., importing, genomics, and proteomics data sets can be provided as archived FASTA files"
- [results] we selected Metaproteomics data, and then Raw FASTA files.: "we selected Metaproteomics data, and then Raw FASTA files."
- [readme] The code is written in Python 3.8.11 and tested on Linux: "The code is written in Python 3.8.11 and tested on Linux"
