---
name: mimic-sequence-data-augmentation
description: Use when training iDeLUCS (or a DeLUCS-style contrastive network) on
  unlabeled DNA sequences in FASTA format and per-sequence augmented training pairs
  are needed to stabilize unsupervised clustering.
when_to_use_negative:
- Input sequences already carry taxonomic labels to be used during training — the
  mimic pipeline is unsupervised; ground-truth labels (GT_file) are only for post
  hoc analysis
- The task requires alignment-based comparison — iDeLUCS clusters via genomic signatures
  without sequence alignment
- batch_sz cannot be set to a multiple of n_mimics on the available machine (batch
  limits may restrict viable mimic counts)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Implements the mimic data augmentation generation (n_mimics parameter) and
    trains the PyTorch contrastive networks on the original–mimic pairs
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: PyTorch
  role: Machine learning framework in which the networks trained on mimic-augmented
    pairs are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:913887c675a83ad15713eed1b3a1cced0f89327caa9263505f26b3302309e88c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - PyTorch
  - Python
  - iDeLUCS
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mimic-sequence-data-augmentation

## Summary

Generates 'mimic' augmented copies of each DNA sequence in a FASTA file so that the iDeLUCS contrastive deep learning network can be trained on original–mimic pairs that share the same genomic k-mer signature. The number of mimics per sequence (n_mimics) directly shapes training pairs, batch composition, and final cluster variance.

## When to use

Use when training iDeLUCS (or a DeLUCS-style contrastive network) on unlabeled DNA sequences in FASTA format and per-sequence augmented training pairs are needed to stabilize unsupervised clustering.

## When NOT to use

- Input sequences already carry taxonomic labels to be used during training — the mimic pipeline is unsupervised; ground-truth labels (GT_file) are only for post hoc analysis
- The task requires alignment-based comparison — iDeLUCS clusters via genomic signatures without sequence alignment
- batch_sz cannot be set to a multiple of n_mimics on the available machine (batch limits may restrict viable mimic counts)

## Inputs

- Single FASTA file of DNA sequences (e.g. Example/Vertebrata.fas)
- Clustering parameters: --k (4, 5, or 6; default 6), --n_mimics (default 50, recommended 50–150), --n_clusters, --batch_sz (must be a multiple of n_mimics)

## Outputs

- n_mimics mimic sequences per input sequence, grouped as original–mimic training pairs
- Per-sequence cluster assignments from the trained network(s), aggregated via the information-theoretic clustering ensemble

## How to apply

Encode each sequence from the input FASTA file as a k-mer frequency vector (k in {4,5,6}, default k=6), then synthesize n_mimics augmented copies per sequence via the iDeLUCS mimic-generation procedure to form original–mimic training pairs. Set n_mimics deliberately: the README default is 50 with a recommended range of 50–150, but published example runs use n_mimics=3 for the Vertebrata dataset (n_epochs=35, lambda=2.8, batch_sz=512) and n_mimics=8 for Actinopterygii. Ensure batch_sz is a multiple of n_mimics, since each batch must receive whole sequence–mimic groups simultaneously. The mimics serve as positive pairs for the contrastive consistency term (lambda=2.8 controls cluster balance) in the PyTorch training loop, reducing variance and improving accuracy over majority-voting aggregation.

## Related tools

- **iDeLUCS** (Implements the mimic data augmentation generation (n_mimics parameter) and trains the PyTorch contrastive networks on the original–mimic pairs) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **PyTorch** (Machine learning framework in which the networks trained on mimic-augmented pairs are implemented)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- The chosen batch_sz is an exact multiple of n_mimics (README constraint)
- Running 'idelucs -h' after installation shows the --n_mimics argument accepted by the CLI
- Post hoc clustering ACC computed against the ground-truth file (e.g. Example/Vertebrata_GT.tsv) with --GT_file improves relative to the majority-voting baseline aggregation
- Number of generated training pairs equals (n_mimics + 1) times the number of input sequences, grouped per original sequence

## Limitations

- A larger batch may improve convergence but can harm accuracy, and batch_sz may be limited by machine capacity (README)
- The latest stable PyTorch release might not be compatible with the local CUDA version, affecting training reproducibility
- n_clusters should be at least n_true_clusters when a GT_file is provided; k is restricted to 4, 5, or 6

## Evidence

- [readme] Mimic count parameter definition: "--n_mimics | integer | Number of data augmentations per sequence that will be considered during training | Default = 50; Recommended Range: 50-150"
- [readme] Batch-multiple constraint: "It must be a multiple of `n_mimcs`."
- [readme] Concrete example invocation with n_mimics: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
- [abstract] Unsupervised, alignment-free scope: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
