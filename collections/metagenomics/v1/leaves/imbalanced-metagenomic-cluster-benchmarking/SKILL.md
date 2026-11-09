---
name: imbalanced-metagenomic-cluster-benchmarking
description: Use when reproducing the simulated metagenomic reads benchmark comparing
  iDeLUCS, DeLUCS, and k-means++ clustering accuracy on unbalanced multi-genome DNA
  read datasets.
when_to_use_negative:
- Datasets with perfectly balanced clusters, where the iDeLUCS advantage from contrastive
  loss on unbalanced clusters is not exercised
- Tasks requiring supervised classification — ground-truth labels in the GT file must
  never be used during training, only for posthoc accuracy alignment
- Datasets needing more than 199 clusters, since n_clusters is limited to the range
  0, 2-199
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: iDeLUCS
  role: Primary deep-learning clustering method under benchmark; PyTorch-backed contrastive
    learning with mimic-based data augmentation
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-clustering baseline run with matched hyperparameters
- name: k-means++
  role: Classical scikit-learn baseline (k=8) applied to the same k-mer signature
    vectors
- name: PyTorch
  role: Machine learning framework used by iDeLUCS for training
- name: Python
  role: Runtime environment for installing iDeLUCS and running all three methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:50ff50a9662fd2f506e76aefc7bbcbff6d52be06aa2671ffa9c70b201d9b657f
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# imbalanced-metagenomic-cluster-benchmarking

## Summary

Benchmark iDeLUCS against DeLUCS and k-means++ on simulated long metagenomic reads (432,333 PacBio reads from 8 microbial genomes) to quantify its advantage on unbalanced clusters, attributed to its clustering ensemble and contrastive loss.

## When to use

Use when reproducing the simulated metagenomic reads benchmark comparing iDeLUCS, DeLUCS, and k-means++ clustering accuracy on unbalanced multi-genome DNA read datasets.

## When NOT to use

- Datasets with perfectly balanced clusters, where the iDeLUCS advantage from contrastive loss on unbalanced clusters is not exercised
- Tasks requiring supervised classification — ground-truth labels in the GT file must never be used during training, only for posthoc accuracy alignment
- Datasets needing more than 199 clusters, since n_clusters is limited to the range 0, 2-199

## Inputs

- PacBio-simulated metagenomic reads FASTA file (432,333 reads, 8 microbial genomes)
- Ground-truth TSV with sequence_id and cluster_id columns
- k-mer-encoded signature vectors (k=6)

## Outputs

- accuracy_comparison.csv with one unsupervised clustering accuracy row per method (iDeLUCS, DeLUCS, k-means++)
- Pairwise accuracy gaps (iDeLUCS minus DeLUCS ~16%, iDeLUCS minus k-means++ ~7%)

## How to apply

Load the PacBio-simulated metagenomic FASTA (432,333 reads, 8 microbial genomes) with its ground-truth TSV containing sequence_id and cluster_id columns, keeping labels for posthoc analysis only. Encode each read as a k-mer frequency signature with k=6 (options 4-6) and run iDeLUCS with n_epochs in the recommended 50-150 range, n_mimics 50-150, batch_sz a multiple of n_mimics, lambda between 1 and 3 for cluster balance, and n_clusters at least the 8 true clusters. Run DeLUCS with matched hyperparameters as the specialized DNA-clustering baseline and k-means++ (scikit-learn, k=8) on the identical signature vectors as the classical baseline. Compute unsupervised clustering accuracy per method by optimally aligning predicted cluster identifiers to ground-truth cluster_id labels, then tabulate the pairwise gaps.

## Related tools

- **iDeLUCS** (Primary deep-learning clustering method under benchmark; PyTorch-backed contrastive learning with mimic-based data augmentation) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-clustering baseline run with matched hyperparameters)
- **k-means++** (Classical scikit-learn baseline (k=8) applied to the same k-mer signature vectors)
- **PyTorch** (Machine learning framework used by iDeLUCS for training)
- **Python** (Runtime environment for installing iDeLUCS and running all three methods)

## Examples

```
idelucs --sequence_file=metagenomic_reads.fas --GT_file=metagenomic_GT.tsv --n_epochs=100 --k=6 --n_clusters=8 --lambda=1.5 --n_mimics=50 --batch_sz=256
```

## Evaluation signals

- accuracy_comparison.csv exists with exactly one accuracy row per method (iDeLUCS, DeLUCS, k-means++)
- Measured iDeLUCS-minus-DeLUCS gap is approximately 16% accuracy
- Measured iDeLUCS-minus-k-means++ gap is approximately 7% accuracy
- n_clusters parameter set to at least the 8 true clusters and batch_sz verified to be a multiple of n_mimics
- Ground-truth cluster_id labels confirmed unused during training and applied only for posthoc optimal label alignment

## Limitations

- The iDeLUCS advantage (~16% over DeLUCS, ~7% over k-means++) is specific to this unbalanced simulated metagenomic dataset and is attributed to the improved ensemble and contrastive loss on unbalanced clusters
- batch_sz may be limited by machine capacity; a larger batch may improve convergence but can harm accuracy
- iDeLUCS' latest stable PyTorch release might not be compatible with your version of CUDA

## Evidence

- [abstract] finding: simulated metagenomic reads accuracy gaps: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [readme] GT file labels posthoc only: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] recommended hyperparameter ranges: "--n_epochs| integer | Number of training epochs. An epoch is defined as a training iteration over all the training pairs.' | Default: `50` ; Recommended Range: `[50,150]`"
- [readme] lambda cluster balance control: "--lambda| float | Hyperparameter to control cluster balance. | Default = 2.8; Recommended Range  1 (highly imbalanced dataset) - 3 (perfectly balanced dataset)"
- [readme] batch size must be multiple of mimics: "It must be a multiple of `n_mimcs`.| Default: 3; Recommended Range: 0-600."
