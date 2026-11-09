---
name: classical-vs-specialized-algorithm-comparison
description: Use when you need to quantify how much a new DNA sequence clustering
  method improves over classical (k-means++, GMM) and DNA-specialized (MeShClust v3.0,
  DeLUCS) baselines on labeled real sequence datasets.
when_to_use_negative:
- The input is a single unlabeled dataset with no ground-truth TSV labels — accuracy
  gaps versus baselines cannot be computed post-hoc
- The sequences are proteins or non-DNA data — the baselines and k-mer genomic signatures
  (k in [4,5,6]) are DNA-specific
- You need supervised classification accuracy rather than unsupervised clustering
  accuracy with optimal cluster-to-label matching
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: iDeLUCS
  role: Deep learning method under evaluation; clusters DNA sequences from genomic
    signatures via CLI
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: MeShClust v3.0
  role: DNA-specialized clustering baseline used for comparison
- name: DeLUCS
  role: DNA-specialized clustering baseline used for comparison
- name: k-means++
  role: Classical clustering baseline run on k-mer genomic signatures with scikit-learn
- name: GMM
  role: 'Classical clustering baseline: Gaussian Mixture Models fit on k-mer genomic
    signatures with scikit-learn'
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:7e58000111dfcc5e3ab434a05e7c8b89086db7384c449fbcb395f986f9f92bea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - GMM
  - MeShClust v3.0
  - PyTorch
  - iDeLUCS
  - k-means++
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# classical-vs-specialized-algorithm-comparison

## Summary

Benchmarks a new DNA-sequence clustering method against two classical clustering baselines (k-means++ and GMM on k-mer genomic signatures) and two DNA-specialized baselines (MeShClust v3.0 and DeLUCS) using unsupervised clustering accuracy on real FASTA datasets with ground-truth TSV labels, then aggregates the average accuracy gaps.

## When to use

Use when you need to quantify how much a new DNA sequence clustering method improves over classical (k-means++, GMM) and DNA-specialized (MeShClust v3.0, DeLUCS) baselines on labeled real sequence datasets.

## When NOT to use

- The input is a single unlabeled dataset with no ground-truth TSV labels — accuracy gaps versus baselines cannot be computed post-hoc
- The sequences are proteins or non-DNA data — the baselines and k-mer genomic signatures (k in [4,5,6]) are DNA-specific
- You need supervised classification accuracy rather than unsupervised clustering accuracy with optimal cluster-to-label matching

## Inputs

- 11 real DNA sequence datasets in FASTA format (e.g. Vertebrata.fas)
- Ground-truth label files in TSV format with sequence_id and cluster_id columns
- k-mer genomic signature representations of each dataset for the scikit-learn classical baselines

## Outputs

- Per-dataset unsupervised clustering accuracy table for all five algorithms (e.g. benchmark_accuracy_results.csv)
- Aggregate accuracy gap summary: mean gap of iDeLUCS vs classical baselines (~20%) and vs specialized baselines (~12%)

## How to apply

Load the 11 real DNA FASTA datasets together with their ground-truth TSV label files (columns sequence_id and cluster_id), keeping labels strictly for post-hoc scoring. Run iDeLUCS via the CLI on each dataset with k-mer length k=6 and --n_clusters set at least to the number of true clusters, using the default lambda=2.8 cluster-balance hyperparameter. Run k-means++ and fit Gaussian Mixture Models on the k-mer genomic signatures with scikit-learn as classical baselines, and run MeShClust v3.0 and DeLUCS as DNA-specialized baselines on the same datasets. Score all five algorithms per dataset with unsupervised clustering accuracy, optimally matching predicted cluster IDs to ground-truth labels, then aggregate per-family averages (e.g. vertebrate mtDNA, Bacteria, viral datasets) and compute iDeLUCS's accuracy gaps versus classical and versus specialized methods. The validation target is the reported finding: iDeLUCS outperforms the classical algorithms by an average of ~20% and the specialized algorithms by an average of ~12%.

## Related tools

- **iDeLUCS** (Deep learning method under evaluation; clusters DNA sequences from genomic signatures via CLI) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **MeShClust v3.0** (DNA-specialized clustering baseline used for comparison)
- **DeLUCS** (DNA-specialized clustering baseline used for comparison)
- **k-means++** (Classical clustering baseline run on k-mer genomic signatures with scikit-learn)
- **GMM** (Classical clustering baseline: Gaussian Mixture Models fit on k-mer genomic signatures with scikit-learn)
- **PyTorch** (Machine learning framework underlying iDeLUCS training)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Per-dataset accuracies are computed with optimal matching of predicted cluster IDs to ground-truth cluster_id labels, so scores are comparable across algorithms
- Aggregate reproduced gaps match the reported finding: ~20% average advantage over classical algorithms and ~12% over specialized algorithms
- iDeLUCS is run with n_clusters at least equal to the number of true clusters and k=6 k-mer length as specified
- Ground-truth labels are used only for post-hoc scoring, never during iDeLUCS training
- The accuracy table and gap summary are written to benchmark_accuracy_results.csv with per-family averages across vertebrate mtDNA, Bacteria, and viral datasets

## Limitations

- The ~20% and ~12% gaps are averages over the specific real DNA datasets analyzed; they may not generalize to other datasets
- iDeLUCS's latest stable PyTorch release might not be compatible with your version of CUDA
- batch_sz must be a multiple of n_mimics and may be limited by machine capacity; larger batches may improve convergence but can harm accuracy

## Evidence

- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ~20%, and the two specialized algorithms by an average of ~12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [abstract] two clustering algorithms specialized in DNA sequences (MeShClust v3.0 and DeLUCS): "two clustering algorithms specialized in DNA sequences (MeShClust v3.0 and DeLUCS)"
- [readme] GT labels used only post-hoc: "These labels won't be used during training, just for posthoc analysis."
- [readme] example CLI invocation with k=6 and lambda=2.8: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
- [readme] n_clusters should be at least n_true_clusters when GT_file is provided: "It should be at least `n_true_clusters` when `GT_file` is provided'."
