---
name: cross-method-clustering-comparison
description: Use when evaluating a new unsupervised DNA sequence clustering tool and
  you need a fair, like-for-like accuracy comparison against specialized (DeLUCS,
  MeShClust v3.0) and classical (k-means++) baselines on unbalanced multi-genome datasets.
when_to_use_negative:
- Sequences require alignment or taxonomic identifiers for the analysis goal — iDeLUCS
  is explicitly an alignment-free method that clusters genomic signatures without
  either
- The dataset is a single-cluster or trivially separable dataset where a hyperparameter-matched
  three-way benchmark adds no information
- You need a clustering method that automatically discovers the number of clusters
  (k-means++ with k=8 and iDeLUCS n_clusters>=8 both require the true cluster count
  to be known or upper-bounded)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Primary deep-learning clustering method under evaluation; interactive unsupervised
    clustering of DNA sequences via genomic signatures
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-sequence clustering baseline run with matched hyperparameters
    on the same k-mer-encoded sequences
- name: k-means++
  role: Classical scikit-learn clustering baseline (k=8) applied to identical k-mer
    signature vectors
- name: PyTorch
  role: Machine learning framework backing iDeLUCS training (contrastive loss, mimic-based
    augmentation)
- name: Python
  role: Runtime environment (pip/venv installation of idelucs and baselines)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:73a042cebcda5c87e28250897f26af8cdf5aa78232541a40f3f0fb85a0226dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - PyTorch
  - Python
  - iDeLUCS
  - k-means++
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-method-clustering-comparison

## Summary

Benchmark a deep-learning DNA sequence clustering method (iDeLUCS) head-to-head against a specialized DNA clustering baseline (DeLUCS) and a classical baseline (k-means++) on identical k-mer-encoded inputs, scoring all methods with unsupervised clustering accuracy against held-out ground truth.

## When to use

Use when evaluating a new unsupervised DNA sequence clustering tool and you need a fair, like-for-like accuracy comparison against specialized (DeLUCS, MeShClust v3.0) and classical (k-means++) baselines on unbalanced multi-genome datasets.

## When NOT to use

- Sequences require alignment or taxonomic identifiers for the analysis goal — iDeLUCS is explicitly an alignment-free method that clusters genomic signatures without either
- The dataset is a single-cluster or trivially separable dataset where a hyperparameter-matched three-way benchmark adds no information
- You need a clustering method that automatically discovers the number of clusters (k-means++ with k=8 and iDeLUCS n_clusters>=8 both require the true cluster count to be known or upper-bounded)

## Inputs

- PacBio-simulated metagenomic FASTA file (432,333 long reads from 8 microbial genomes)
- Ground-truth TSV with sequence_id and cluster_id columns (labels held out from training)
- k-mer frequency signature vectors (k=6) shared by all three methods

## Outputs

- accuracy_comparison.csv with one unsupervised clustering accuracy row per method (iDeLUCS, DeLUCS, k-means++)
- Pairwise accuracy gaps: iDeLUCS-minus-DeLUCS (~16%) and iDeLUCS-minus-k-means++ (~7%)

## How to apply

Encode every read from a single FASTA file as a k-mer frequency signature with k=6 (options are 4, 5, or 6 per the iDeLUCS README), then run iDeLUCS (PyTorch-backed, contrastive learning with mimic-based data augmentation) using n_epochs in the recommended 50-150 range, n_mimics 50-150, batch_sz a multiple of n_mimics, lambda between 1 and 3 (lower for highly imbalanced datasets), and n_clusters set to at least the number of true clusters (8 here). Run DeLUCS on the identical k-mer-encoded sequences with matched hyperparameters as the specialized DNA-clustering baseline, and scikit-learn k-means++ (k=8) on the same signature vectors as the classical baseline. Ground-truth TSV labels (sequence_id and cluster_id columns) are reserved for posthoc analysis only and never used in training. Compute unsupervised clustering accuracy by optimally aligning predicted cluster identifiers to ground-truth cluster_id, then tabulate accuracies and pairwise gaps. On the PacBio-simulated metagenomic dataset (432,333 reads, 8 microbial genomes), verify iDeLUCS exceeds DeLUCS by ~16% and k-means++ by ~7%, attributed to the improved clustering ensemble and contrastive loss on unbalanced clusters.

## Related tools

- **iDeLUCS** (Primary deep-learning clustering method under evaluation; interactive unsupervised clustering of DNA sequences via genomic signatures) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-sequence clustering baseline run with matched hyperparameters on the same k-mer-encoded sequences)
- **k-means++** (Classical scikit-learn clustering baseline (k=8) applied to identical k-mer signature vectors)
- **PyTorch** (Machine learning framework backing iDeLUCS training (contrastive loss, mimic-based augmentation))
- **Python** (Runtime environment (pip/venv installation of idelucs and baselines))

## Examples

```
idelucs --sequence_file=simulated_metagenomic.fas --GT_file=simulated_metagenomic_GT.tsv --n_epochs=100 --lambda=2 --k=6 --n_clusters=8 --n_mimics=50 --batch_sz=512
```

## Evaluation signals

- accuracy_comparison.csv exists and contains exactly one accuracy row per method (iDeLUCS, DeLUCS, k-means++)
- Measured iDeLUCS-minus-DeLUCS gap falls at approximately 16% on the simulated metagenomic reads dataset
- Measured iDeLUCS-minus-k-means++ gap falls at approximately 7% on the same dataset
- All three methods consumed identical k=6 signature vectors and iDeLUCS hyperparameters stay in the recommended ranges (n_epochs 50-150, n_mimics 50-150, lambda 1-3, batch_sz multiple of n_mimics)
- Ground-truth cluster_id labels were used only for posthoc optimal label alignment, never during training

## Limitations

- batch_sz is limited by machine capacity and a too-large batch may harm accuracy; it must be a multiple of n_mimics
- n_clusters must be at least n_true_clusters when a GT file is provided; automatic cluster-number discovery requires n_clusters=0 (iDeLUCS + HDBSCAN)
- The latest stable PyTorch release may not be compatible with the user's CUDA version; PyQt5 pip distribution does not support Apple Silicon for the GUI
- Reported advantages (~12% over specialized algorithms, ~20% over classical on real datasets) vary by dataset; the ~16%/~7% gaps are specific to this unbalanced simulated metagenomic benchmark

## Evidence

- [results] iDeLUCS outperforms two classical clustering algorithms and two specialized algorithms in unsupervised clustering accuracy on real DNA sequence datasets: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [readme] Ground-truth labels are excluded from training and used only posthoc: "These labels won't be used during training, just for posthoc analysis."
- [readme] k-mer encoding options for the signature vectors shared by all methods: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [abstract] iDeLUCS framework and dataset scope including simulated metagenomic reads: "several real genomic datasets from organisms in kingdoms Animalia, Protista, Fungi, Bacteria, and Archaea, three datasets of viral genomes, a dataset of simulated metagenomic reads from microbial"
- [readme] Reference CLI invocation with ground truth and hyperparameters: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
