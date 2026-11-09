---
name: unsupervised-dna-sequence-clustering
description: Use when clustering unlabeled DNA sequences from a single FASTA file
  into known or estimated clusters without alignment or taxonomic identifiers, e.g.
  mitochondrial genomes, bacterial, or viral datasets with 2-199 expected clusters.
when_to_use_negative:
- Datasets with average sequence length below 500 bp or more than 200 expected clusters
  — these regimes were explicitly excluded from the evaluation and flagged as future
  work
- Datasets where the number of true clusters is unknown and fine-grained automatic
  cluster discovery is required, unless willing to use n_clusters=0 (iDeLUCS+HDBSCAN
  mode)
- Tasks requiring aligned phylogenetic placement or taxonomic classification with
  reference databases rather than unsupervised clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Primary deep learning clustering tool; detects genomic signatures and clusters
    DNA sequences without alignment
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: DNA-specialized baseline algorithm for accuracy comparison
- name: MeShClust v3.0
  role: DNA-specialized baseline run in automatic and optimized identity-threshold
    modes
- name: PyTorch
  role: Machine learning development framework underlying iDeLUCS training
- name: k-means++
  role: Classical baseline fitted on k-mer genomic signatures via scikit-learn
- name: GMM
  role: Gaussian Mixture Model classical baseline fitted on k-mer genomic signatures
    via scikit-learn
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:461a632bab5a5583202ad6740a13f1d63c2f47d83badbbc19ca3d9ca4f45ec1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - GMM
  - MeShClust v3.0
  - PyTorch
  - Python
  - iDeLUCS
  - k-means++
  - pip
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# unsupervised-dna-sequence-clustering

## Summary

Cluster DNA sequences in an alignment-free, unsupervised manner using iDeLUCS, a PyTorch-based deep learning tool that detects genomic signatures (k-mer frequencies) and clusters sequences without alignment or taxonomic identifiers, benchmarked against k-means++, GMM, DeLUCS, and MeShClust v3.0.

## When to use

Use when clustering unlabeled DNA sequences from a single FASTA file into known or estimated clusters without alignment or taxonomic identifiers, e.g. mitochondrial genomes, bacterial, or viral datasets with 2-199 expected clusters.

## When NOT to use

- Datasets with average sequence length below 500 bp or more than 200 expected clusters — these regimes were explicitly excluded from the evaluation and flagged as future work
- Datasets where the number of true clusters is unknown and fine-grained automatic cluster discovery is required, unless willing to use n_clusters=0 (iDeLUCS+HDBSCAN mode)
- Tasks requiring aligned phylogenetic placement or taxonomic classification with reference databases rather than unsupervised clustering

## Inputs

- Single FASTA file of DNA sequences (e.g. Vertebrata.fas)
- Optional ground-truth TSV file with sequence_id and cluster_id columns (and optional phylogeny column) for posthoc analysis only
- Hyperparameters: k-mer length k in [4,5,6], n_clusters (2-199, or 0 for iDeLUCS+HDBSCAN), lambda (default 2.8), n_epochs (50-150), n_mimics, batch_sz (multiple of n_mimics)

## Outputs

- Cluster identifier assignment for every sequence in the input FASTA file
- Posthoc evaluation metrics: Davies-Bouldin index, silhouette score, Homogeneity, Completeness, and clustering accuracy (ACC)
- Results tables aggregating per-dataset metrics (e.g. table2_clustering_metrics.csv, benchmark_accuracy_results.csv)

## How to apply

Run iDeLUCS with k-mer length k=6 (options [4,5,6]) and lambda=2.8 to control cluster balance, setting --n_clusters to at least the true number of clusters when ground truth is available. Provide --GT_file (TSV with sequence_id and cluster_id columns) strictly for posthoc scoring — ground-truth labels are never used during training. Choose --n_epochs within the recommended 50-150 range and pick --batch_sz as a multiple of --n_mimics. Compare against classical baselines (k-means++ and GMM on k-mer genomic signatures via scikit-learn) and DNA-specialized baselines (DeLUCS, MeShClust v3.0 in automatic and optimized identity-threshold modes). Evaluate with intrinsic metrics (Davies-Bouldin index, silhouette score) and external metrics (Homogeneity, Completeness, ACC) by optimally matching predicted cluster IDs to ground-truth cluster_id labels. iDeLUCS reported ~20% average ACC improvement over classical algorithms and ~12% over specialized ones on real DNA datasets, and 78-89.7% accuracy on new mitochondrial datasets (Protista, Insecta, Fungi).

## Related tools

- **iDeLUCS** (Primary deep learning clustering tool; detects genomic signatures and clusters DNA sequences without alignment) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (DNA-specialized baseline algorithm for accuracy comparison)
- **MeShClust v3.0** (DNA-specialized baseline run in automatic and optimized identity-threshold modes)
- **PyTorch** (Machine learning development framework underlying iDeLUCS training)
- **k-means++** (Classical baseline fitted on k-mer genomic signatures via scikit-learn)
- **GMM** (Gaussian Mixture Model classical baseline fitted on k-mer genomic signatures via scikit-learn)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- External metrics (Homogeneity, Completeness, ACC) computed against ground-truth cluster_id labels must show iDeLUCS outperforming classical baselines by ~20% and specialized baselines by ~12% on average on real DNA datasets
- On the new mitochondrial datasets (Protista, Insecta, Fungi), iDeLUCS unsupervised ACC should fall in the 78-89.7% range, outperforming all baselines on Insects and Fungi and performing comparably on Protist
- The results table must contain one row per method x dataset cell with external metrics computed against cluster_id for all five methods on all three datasets
- batch_sz must be a multiple of n_mimics; n_clusters must be at least n_true_clusters when a GT_file is provided
- Verify ground-truth labels were used only posthoc and never for training

## Limitations

- Datasets with average sequence length < 500 bp or more than 200 clusters were excluded from evaluation; systematic testing in these regimes is explicitly future work
- n_clusters valid range is 2-199 (or 0 for automatic HDBSCAN mode)
- PyTorch latest stable release may not be compatible with your CUDA version; batch_sz may be limited by machine capacity
- PyQt5 pip distribution does not support Apple Silicon Macs; a brew-based symbolic link workaround is needed for the GUI

## Evidence

- [abstract] iDeLUCS outperforms classical vs specialized baselines: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [readme] GT labels posthoc only: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] k-mer options and balance lambda: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [readme] README example invocation with ground truth: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
- [readme] batch_sz constraint: "It must be a multiple of `n_mimcs`.| Default: 3; Recommended Range: 0-600."
