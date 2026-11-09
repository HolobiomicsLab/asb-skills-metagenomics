---
name: contrastive-consistency-training-of-clustering-networks
description: Use when clustering unlabeled DNA sequences alignment-free with a deep
  learning model, needing mimic-based augmentation pairs and a contrastive consistency
  term to reduce variance before a clustering ensemble aggregates assignments.
when_to_use_negative:
- Sequences require alignment or taxonomic identifiers for the analysis — iDeLUCS
  is explicitly alignment-free and identifier-free
- 'You need a supervised classifier: GT labels must never be used during training,
  only for post hoc ACC evaluation'
- The dataset is known to be highly imbalanced and you cannot tune lambda (recommended
  range 1-3) toward 1
edam_operation: http://edamontology.org/operation_3459
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: CLI/GUI tool implementing the contrastive-consistency training pipeline for
    unsupervised clustering of DNA sequences
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: PyTorch
  role: machine learning development framework used to train the clustering networks
    with the contrastive consistency term
- name: pip
  role: installs the idelucs package (pip install idelucs) required to run the pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:32d72dedc3b1696ed1de15de083993df87ba5bdd1dceaea9146fb8c68832c183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - PyTorch
  - Python
  - iDeLUCS
  - pip
  edam_operation: http://edamontology.org/operation_3459
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# contrastive-consistency-training-of-clustering-networks

## Summary

Train multiple PyTorch networks on k-mer frequency vectors of DNA sequences with mimic-sequence augmentations and a contrastive consistency term (lambda controls cluster balance), as done in iDeLUCS for alignment-free unsupervised clustering of genomic sequences.

## When to use

Use when clustering unlabeled DNA sequences alignment-free with a deep learning model, needing mimic-based augmentation pairs and a contrastive consistency term to reduce variance before a clustering ensemble aggregates assignments.

## When NOT to use

- Sequences require alignment or taxonomic identifiers for the analysis — iDeLUCS is explicitly alignment-free and identifier-free
- You need a supervised classifier: GT labels must never be used during training, only for post hoc ACC evaluation
- The dataset is known to be highly imbalanced and you cannot tune lambda (recommended range 1-3) toward 1

## Inputs

- Single FASTA file of DNA sequences (e.g. Example/Vertebrata.fas)
- Optional TSV ground-truth file with columns sequence_id and cluster_id (e.g. Example/Vertebrata_GT.tsv), used only post hoc
- Hyperparameters: k (4-6, default 6), n_clusters, n_epochs (default 50), n_mimics, batch_sz, lambda (default 2.8), optimizer, scheduler

## Outputs

- Per-network cluster assignments for every sequence in the FASTA file
- Ensemble-aggregated cluster identifiers (information-theoretic clustering ensemble)
- Post hoc unsupervised clustering ACC when a GT file is provided (e.g. written to acc_comparison.tsv for ensemble vs majority-voting baseline)

## How to apply

Encode each sequence from the input FASTA as a k-mer frequency vector (k in {4,5,6}, default 6) and generate mimic-sequence data augmentations per sequence (e.g. n_mimics=3 with batch_sz=512) to form training pairs. Train multiple PyTorch networks with the added contrastive consistency term in the loss function, setting lambda=2.8 to control cluster balance (range roughly 1 for imbalanced to 3 for perfectly balanced datasets) and n_epochs=35 for the Vertebrata example. The GT file labels are never used during training, only for post hoc analysis, so training remains unsupervised. Aggregate per-network cluster assignments with the information-theoretic clustering ensemble (replacing the majority voting of the predecessor DeLUCS), then compute unsupervised clustering ACC post hoc against ground-truth labels. The contrastive term plus the ensemble together reduce variance and boost accuracy over the baseline.

## Related tools

- **iDeLUCS** (CLI/GUI tool implementing the contrastive-consistency training pipeline for unsupervised clustering of DNA sequences) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **PyTorch** (machine learning development framework used to train the clustering networks with the contrastive consistency term)
- **pip** (installs the idelucs package (pip install idelucs) required to run the pipeline)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Post hoc unsupervised clustering ACC for the ensemble with the contrastive term must exceed the majority-voting baseline ACC on the Vertebrata ground-truth labels
- batch_sz must be a multiple of n_mimics or training will be misconfigured
- n_clusters must be at least the number of true clusters when a GT_file is provided
- Reported improvement: iDeLUCS outperforms two classical clustering algorithms by ~20% and two specialized algorithms by ~12% in unsupervised clustering accuracy
- Model output assigns exactly one cluster identifier to every sequence in the input FASTA file

## Limitations

- PyTorch version is not specified; its latest stable release might not be compatible with your version of CUDA
- GT file labels must have sequence_id values matching the FASTA identifiers, otherwise post hoc ACC cannot be computed
- Larger batch sizes may improve convergence but may harm accuracy, and batch_sz is limited by machine capacity
- k-mer length is restricted to options 4, 5, or 6

## Evidence

- [abstract] contrastive consistency and clustering ensemble: "detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
- [abstract] outperformance: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [readme] GT labels post hoc only: "These labels won't be used during training, just for posthoc analysis."
- [readme] lambda parameter: "--lambda| float | Hyperparameter to control cluster balance."
- [methods] PyTorch framework: "iDeLUCS uses PyTorch as the machine learning development framework"
