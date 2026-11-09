---
name: kmer-frequency-encoding-of-dna-sequences
description: Use when raw DNA sequences in a single FASTA file must be converted to
  numeric feature vectors for alignment-free unsupervised clustering, e.g., as the
  encoding step before training iDeLUCS networks on genomic signatures.
when_to_use_negative:
- The data are already numeric feature vectors or embeddings rather than raw DNA sequences
  in FASTA format
- An alignment-based analysis is required (this encoding intentionally discards alignment;
  iDeLUCS clusters 'without the need for sequence alignment or taxonomic identifiers')
- The desired k-mer length falls outside iDeLUCS's supported options of 4, 5, or 6
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: CLI/GUI tool that performs the k-mer frequency encoding (--k parameter, default
    6, options 4/5/6) and downstream clustering of the encoded sequences
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: PyTorch
  role: Machine learning framework in which the networks consuming the encoded k-mer
    frequency vectors are trained
- name: pip
  role: Used to install the idelucs package (pip install idelucs) that provides the
    encoding and clustering functionality
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:de570056c73c8aac3d34f77462f8a1b285f78a79e8bed064318497b13f03e399
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - PyTorch
  - Python
  - iDeLUCS
  - pip
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# k-mer frequency encoding of DNA sequences

## Summary

Encodes each DNA sequence from a FASTA file as a k-mer frequency vector (k=6 default in iDeLUCS, options k=4,5,6) so that alignment-free deep learning can cluster sequences by genomic signatures. This numeric representation is the first step of the iDeLUCS pipeline and replaces the need for sequence alignment or taxonomic identifiers.

## When to use

Use when raw DNA sequences in a single FASTA file must be converted to numeric feature vectors for alignment-free unsupervised clustering, e.g., as the encoding step before training iDeLUCS networks on genomic signatures.

## When NOT to use

- The data are already numeric feature vectors or embeddings rather than raw DNA sequences in FASTA format
- An alignment-based analysis is required (this encoding intentionally discards alignment; iDeLUCS clusters 'without the need for sequence alignment or taxonomic identifiers')
- The desired k-mer length falls outside iDeLUCS's supported options of 4, 5, or 6

## Inputs

- Single FASTA file of DNA sequences (e.g., Example/Vertebrata.fas)
- Optional tab-separated ground-truth file with columns sequence_id and cluster_id (e.g., Example/Vertebrata_GT.tsv), used only post hoc

## Outputs

- Per-sequence k-mer frequency vectors (k=6 by default) used as network input features
- Cluster identifier assignment for every sequence in the FASTA file
- Post hoc unsupervised clustering ACC against ground-truth labels when a GT file is provided

## How to apply

Load the sequences from the input FASTA file (e.g., Example/Vertebrata.fas) and encode each sequence as a k-mer frequency vector using iDeLUCS's --k parameter, which accepts k=4, 5, or 6 with a default of k=6. The chosen k defines the length of the substring signature whose frequencies form the vector, capturing genomic signatures without alignment. These vectors then feed downstream steps: mimic-sequence augmentations (n_mimics) form training pairs for PyTorch networks, and final cluster assignments come from the information-theoretic clustering ensemble. If ground-truth labels are available (e.g., Example/Vertebrata_GT.tsv with sequence_id and cluster_id columns), they are used only for post hoc analysis, never during training.

## Related tools

- **iDeLUCS** (CLI/GUI tool that performs the k-mer frequency encoding (--k parameter, default 6, options 4/5/6) and downstream clustering of the encoded sequences) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **PyTorch** (Machine learning framework in which the networks consuming the encoded k-mer frequency vectors are trained)
- **pip** (Used to install the idelucs package (pip install idelucs) that provides the encoding and clustering functionality)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Every sequence in the FASTA file receives exactly one cluster identifier assignment
- The --k value used is within iDeLUCS's allowed set [4, 5, 6] (default 6)
- When a GT file is supplied, post hoc unsupervised clustering ACC is computed against the cluster_id labels and the information-theoretic ensemble ACC exceeds the majority-voting baseline ACC (e.g., on Vertebrata.fas with k=6, n_mimics=3, lambda=2.8, n_epochs=35, batch_sz=512)
- batch_sz is a multiple of n_mimics as required by the training step consuming the encoded vectors
- Installation is verified via `idelucs -h` before encoding runs

## Limitations

- k-mer length is restricted to 4, 5, or 6; other k values are not supported
- n_clusters must be within 2-199 (or 0 for iDeLUCS + HDBSCAN automatic detection) and should be at least the true number of clusters when a GT file is provided
- iDeLUCS uses PyTorch, whose latest stable release may not be compatible with your CUDA version; batch size may also be limited by machine capacity
- The GUI depends on PyQt5, which the pip distribution does not support on Apple Silicon without a homebrew workaround

## Evidence

- [readme] k-mer length, default and options: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [abstract] alignment-free genomic signature clustering: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
- [readme] example encoding/clustering invocation with ground truth: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
- [readme] GT labels are post hoc only: "These labels won't be used during training, just for posthoc analysis."
