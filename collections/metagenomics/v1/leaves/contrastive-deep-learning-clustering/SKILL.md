---
name: contrastive-deep-learning-clustering
description: Use when the goal is unsupervised clustering of unlabeled DNA sequences
  in a single FASTA file without alignment or taxonomic identifiers, especially for
  large, diverse, or unbalanced datasets (e.g. simulated metagenomic reads).
when_to_use_negative:
- Input is not raw DNA sequences but an existing feature table or precomputed embedding
  — iDeLUCS expects a single FASTA of nucleotide sequences and builds k-mer signatures
  itself.
- The number of clusters is known to exceed 199 — the n_clusters parameter is limited
  to the range 0, 2-199.
- The analysis requires supervised classification with labeled training data — iDeLUCS
  is unsupervised; GT labels are never used during training, only for posthoc analysis.
edam_operation: http://edamontology.org/operation_3459
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: iDeLUCS
  role: Primary contrastive-learning clustering tool; assigns cluster identifiers
    to all DNA sequences in a FASTA file, with optional HDBSCAN auto-k mode (n_clusters=0)
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-clustering baseline run with matched hyperparameters to quantify
    iDeLUCS's accuracy gain
- name: k-means++
  role: Classical baseline run on the identical k-mer signature vectors for accuracy
    comparison
- name: MeShClust v3.0
  role: Specialized DNA clustering algorithm whose reported 99.3% average accuracy
    serves as the reference on the synthetic benchmark
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training; version must be compatible
    with the local CUDA installation
- name: HDBSCAN
  role: Used with iDeLUCS (n_clusters=0) to automatically determine the number of
    clusters
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:0e9ddb56836c3205eb8b74b45de207199c90b8c8d54e7ed2b4a9df8dc9aa7d22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - HDBSCAN
  - MeShClust v3.0
  - PyTorch
  - Python
  - iDeLUCS
  - k-means++
  edam_operation: http://edamontology.org/operation_3459
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# contrastive-deep-learning-clustering

## Summary

Apply iDeLUCS, a PyTorch-based unsupervised deep learning tool, to cluster DNA sequences alignment-free by detecting genomic signatures, using mimic-based data augmentation and a contrastive loss with a lambda term that controls cluster balance.

## When to use

Use when the goal is unsupervised clustering of unlabeled DNA sequences in a single FASTA file without alignment or taxonomic identifiers, especially for large, diverse, or unbalanced datasets (e.g. simulated metagenomic reads).

## When NOT to use

- Input is not raw DNA sequences but an existing feature table or precomputed embedding — iDeLUCS expects a single FASTA of nucleotide sequences and builds k-mer signatures itself.
- The number of clusters is known to exceed 199 — the n_clusters parameter is limited to the range 0, 2-199.
- The analysis requires supervised classification with labeled training data — iDeLUCS is unsupervised; GT labels are never used during training, only for posthoc analysis.

## Inputs

- Single FASTA file of DNA sequences (e.g. Example/Vertebrata.fas)
- Optional ground-truth TSV with sequence_id and cluster_id columns (optionally a phylogeny column), used posthoc only
- Hyperparameters: k (4-6), n_clusters (0 or 2-199), n_epochs (50-150), n_mimics (50-150), batch_sz (multiple of n_mimics), lambda (1-3), optimizer ['SGD','Adam','RMSprop'], scheduler ['None','Plateau','Triangle']

## Outputs

- Cluster identifier assignments for every sequence in the input FASTA file
- Posthoc unsupervised clustering accuracy (ACC) values when a GT file is supplied
- Comparison tables of per-method ACC (e.g. accuracy_comparison.csv) versus DeLUCS, k-means++, and MeShClust v3.0

## How to apply

Encode each sequence as a k-mer frequency signature with k between 4 and 6 (default 6), then train iDeLUCS with n_epochs in the recommended 50-150 range, n_mimics 50-150, batch_sz a multiple of n_mimics, and lambda between 1 (highly imbalanced) and 3 (perfectly balanced) to control cluster balance via the contrastive loss. Set n_clusters to at least the number of true clusters when a GT file is provided; set n_clusters=0 to launch iDeLUCS + HDBSCAN to automatically determine the number of clusters. Ground-truth labels in the TSV (columns sequence_id and cluster_id) are never used during training, only for posthoc analysis via optimal label alignment to compute unsupervised clustering accuracy (ACC). Compare against baselines such as DeLUCS, k-means++ (scikit-learn), and MeShClust v3.0 on identical signature vectors, e.g. iDeLUCS reaches ~16% higher accuracy than DeLUCS and ~7% higher than k-means++ on simulated metagenomic reads, and ~98.5% average ACC on 12 synthetic datasets with given k (97.3% with HDBSCAN auto-k).

## Related tools

- **iDeLUCS** (Primary contrastive-learning clustering tool; assigns cluster identifiers to all DNA sequences in a FASTA file, with optional HDBSCAN auto-k mode (n_clusters=0)) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-clustering baseline run with matched hyperparameters to quantify iDeLUCS's accuracy gain)
- **k-means++** (Classical baseline run on the identical k-mer signature vectors for accuracy comparison)
- **MeShClust v3.0** (Specialized DNA clustering algorithm whose reported 99.3% average accuracy serves as the reference on the synthetic benchmark)
- **PyTorch** (Machine learning framework underlying iDeLUCS training; version must be compatible with the local CUDA installation)
- **HDBSCAN** (Used with iDeLUCS (n_clusters=0) to automatically determine the number of clusters)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- accuracy_comparison.csv exists with one accuracy row per method, and iDeLUCS-minus-baseline gaps on the metagenomic dataset fall at approximately 16% (vs DeLUCS) and approximately 7% (vs k-means++)
- On the 12 Girgis (2022) synthetic datasets, reproduced averages are approximately 98.5% ACC with the number of clusters given and 97.3% with HDBSCAN auto-k, alongside the 99.3% MeShClust v3.0 reference
- GT file conforms to schema: sequence_id values match the FASTA identifiers and a cluster_id column is present; labels are confirmed unused during training
- batch_sz is a multiple of n_mimics and n_clusters is at least n_true_clusters when a GT file is provided
- Installation test passes: `idelucs -h` runs without error

## Limitations

- iDeLUCS uses PyTorch, whose latest stable release might not be compatible with your version of CUDA
- batch_sz might be limited by the capacity of the machine; a larger batch may improve convergence but may harm accuracy
- The pip distribution of PyQt5 does not support Apple Silicon; GUI users on Apple Silicon must link the brew-installed PyQt5 into the virtualenv
- GUI executables are hosted outside a standard repository (vault.cs.uwaterloo.ca) and Qt platform plugins must be installed

## Evidence

- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [methods] GT labels never used during training: "These labels won't be used during training, just for posthoc analysis."
- [readme] k-mer length options 4-6: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [readme] n_clusters=0 launches iDeLUCS + HDBSCAN auto-k: "Use 0 to launch iDeLCUS + HDBSCAn to automatically find the number of clusters in fine-grained clustering."
- [readme] lambda controls cluster balance 1-3: "--lambda| float | Hyperparameter to control cluster balance. | Default = 2.8; Recommended Range  1 (highly imbalanced dataset) - 3 (perfectly balanced dataset)"
