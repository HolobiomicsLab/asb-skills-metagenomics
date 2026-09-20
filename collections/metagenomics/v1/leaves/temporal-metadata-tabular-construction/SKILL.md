---
name: temporal-metadata-tabular-construction
description: Use when raw archived FASTA files must be converted into a numeric feature
  table with an embedded temporal column so downstream MOVIS clustering or visualization
  steps can operate on sequence data.
when_to_use_negative:
- Input is already a tabular data set (CSV/TSV) — MOVIS's filtering step applies only
  to tabular data, and FASTA embedding is unnecessary
- The goal is physico-chemical FASTA features (e.g., the 43 proteomics features such
  as Molecular Weight, Isoelectric Point, Instability Index) rather than Word2Vec
  embedding vectors
- Input is raw microarray-based data, which MOVIS explicitly cannot accept directly
  and requires preprocessing first
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: MOVIS
  role: Host pipeline whose data analysis core consumes the constructed 101-column
    embedded table for clustering (K-Means, Elbow rule) and visualization
  repo: https://github.com/AAnzel/MOVIS
- name: Biopython
  role: SeqIO parsing of each contained sequence from the archived FASTA files
- name: Gensim
  role: Word2Vec model trained on k-mer tokens to produce the 100-dimensional per-sequence
    embeddings
- name: Pandas
  role: Assembly of the 101-column DataFrame and writing of embeddings.csv
- name: Python
  role: Implementation language of the embedding module (article reports Python 3.8.11)
  repo: https://github.com/AAnzel/MOVIS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.02.012
    title: 'MOVIS: A multi-omics software solution for multi-modal time-series clustering,
      embedding, and visualizing tasks'
  source_content_hash: sha256:b4a6a367b9200be1d4bdaf20e4b835d98833e6b389626e80bf3fbde785462631
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# temporal-metadata-tabular-construction

## Summary

Constructs a single tabular data set (embeddings.csv) in which each archived FASTA file from the Example 1 metaproteomics data set is reduced to one row combining its temporal information with a 100-dimensional Word2Vec embedding vector, making sequence data directly consumable by MOVIS's clustering (e.g., K-Means with Elbow rule) and visualization steps.

## When to use

Use when raw archived FASTA files must be converted into a numeric feature table with an embedded temporal column so downstream MOVIS clustering or visualization steps can operate on sequence data.

## When NOT to use

- Input is already a tabular data set (CSV/TSV) — MOVIS's filtering step applies only to tabular data, and FASTA embedding is unnecessary
- The goal is physico-chemical FASTA features (e.g., the 43 proteomics features such as Molecular Weight, Isoelectric Point, Instability Index) rather than Word2Vec embedding vectors
- Input is raw microarray-based data, which MOVIS explicitly cannot accept directly and requires preprocessing first

## Inputs

- Archived (zipped) raw FASTA files from the Example 1 metaproteomics data set (Herold et al. BWWTP multi-metaomics data)
- Temporal metadata (sampling time points) associated with each FASTA file

## Outputs

- embeddings.csv: a Pandas DataFrame written as CSV with exactly 101 columns (1 temporal + 100 Word2Vec embedding dimensions) and one row per input FASTA file

## How to apply

Parse each FASTA file contained in the archived metaproteomics data set using Biopython's SeqIO. Tokenize every nucleotide/amino-acid sequence into overlapping k-mer tokens and train a Gensim Word2Vec model on this token corpus. Use the trained model to embed each parsed sequence as a 100-dimensional vector, then average all per-sequence vectors within each FASTA file to obtain one representative vector per file. Assemble a Pandas DataFrame with one row per FASTA file and 101 columns — one column holding the temporal information plus the 100 embedding-dimension columns — and write it to embeddings.csv. This mirrors the article's description of a Word2Vec-embedded tabular data set with 'the same number of rows as FASTA files in an archived dataset and 101 columns', where the temporal column preserves the time-series dimension required by MOVIS's time-series analysis core.

## Related tools

- **MOVIS** (Host pipeline whose data analysis core consumes the constructed 101-column embedded table for clustering (K-Means, Elbow rule) and visualization) — https://github.com/AAnzel/MOVIS
- **Biopython** (SeqIO parsing of each contained sequence from the archived FASTA files)
- **Gensim** (Word2Vec model trained on k-mer tokens to produce the 100-dimensional per-sequence embeddings)
- **Pandas** (Assembly of the 101-column DataFrame and writing of embeddings.csv)
- **Python** (Implementation language of the embedding module (article reports Python 3.8.11)) — https://github.com/AAnzel/MOVIS

## Examples

```
from Bio import SeqIO; seqs=[list(str(r.seq)) for r in SeqIO.parse('sample.fasta','fasta')]; from gensim.models import Word2Vec; model=Word2Vec(seqs, vector_size=100)
```

## Evaluation signals

- embeddings.csv exists after the module runs
- embeddings.csv has exactly 101 columns (1 temporal + 100 embedding dimensions)
- Row count of embeddings.csv equals the number of FASTA files in the input archive
- Each cell in the 100 embedding columns is a valid float equal to the average of that file's per-sequence Word2Vec vectors (no NaNs from unparseable files)
- Temporal column values correctly correspond to each FASTA file's sampling time point in the Example 1 metaproteomics series

## Limitations

- MOVIS presupposes data are preprocessed — quality controlled and filtered — before entering the data analysis core
- No taxa linking is enabled for metagenomics data; MOVIS is not explicitly designed to work with meta-omics data
- Direct integrative combination of multiple omic types is planned but not yet available in MOVIS
- Only 100 embedding dimensions are produced (101 columns with temporal metadata); other dimensionalities fall outside the validated schema

## Evidence

- [methods] Word2Vec embedding produces a tabular dataset with one row per FASTA file and 101 columns: "a tabular data set containing the same number of rows as FASTA files in an archived dataset and 101 columns"
- [full_text] Embedding output feeds clustering with Elbow rule: [evidence span withheld — anchor status: not_found]
- [readme] Dependencies pin the embedding stack versions: "|biopython|1.78|
|gensim|4.0.1|
|numpy|1.21.2|
|pandas|1.3.5|"
- [results] Raw FASTA selected from Example 1 metaproteomics data: "we selected Metaproteomics data, and then Raw FASTA files."
