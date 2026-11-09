---
name: kmer-genomic-signature-clustering
description: Use when clustering unaligned DNA FASTA sequences into expected clusters
  without taxonomic identifiers, e.g. mitochondrial (Vertebrata, Protista, Insecta,
  Fungi), Bacterial, Archaeal, or viral genomes, and comparing against classical and
  DNA-specialized baselines.
when_to_use_negative:
- Datasets with average sequence length below 500 bp or expected cluster counts above
  199 (e.g. ~5000) — these regimes were excluded from the evaluation and flagged as
  future work, so the method is unvalidated there
- Datasets requiring aligned sequences or taxonomic identifiers as clustering input
  — iDeLUCS is explicitly alignment-free and identifier-free
- Fully supervised classification where labeled training data can be used during training
  — the GT labels here are posthoc-only
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: iDeLUCS
  role: 'Primary tool: deep-learning detection of genomic signatures and unsupervised
    clustering of DNA sequences'
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: PyTorch
  role: Machine learning development framework underlying iDeLUCS training
- name: DeLUCS
  role: DNA-specialized clustering baseline for accuracy comparison
- name: MeShClust v3.0
  role: DNA-specialized clustering baseline, run in automatic and optimized identity-threshold
    modes
- name: k-means++
  role: Classical baseline clustering on k-mer genomic signatures
- name: GMM
  role: Classical baseline (Gaussian Mixture Model) on k-mer genomic signatures
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:1cf1eb67c1491d0c2cabcfc60b1866e58118978f4f174f587530a9b09c17e46d
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# kmer-genomic-signature-clustering

## Summary

Cluster unlabeled DNA sequences alignment-free by detecting k-mer-based genomic signatures and unsupervised deep learning, as implemented in iDeLUCS (Bioinformatics, 2023), and score clusters against ground-truth cluster_id labels post hoc.

## When to use

Use when clustering unaligned DNA FASTA sequences into expected clusters without taxonomic identifiers, e.g. mitochondrial (Vertebrata, Protista, Insecta, Fungi), Bacterial, Archaeal, or viral genomes, and comparing against classical and DNA-specialized baselines.

## When NOT to use

- Datasets with average sequence length below 500 bp or expected cluster counts above 199 (e.g. ~5000) — these regimes were excluded from the evaluation and flagged as future work, so the method is unvalidated there
- Datasets requiring aligned sequences or taxonomic identifiers as clustering input — iDeLUCS is explicitly alignment-free and identifier-free
- Fully supervised classification where labeled training data can be used during training — the GT labels here are posthoc-only

## Inputs

- Single FASTA file of DNA sequences (e.g. Vertebrata.fas, Actinopterygii.fas)
- Ground-truth tab-separated label file with 'sequence_id' and 'cluster_id' columns (optional 'phylogeny' column) for posthoc scoring only

## Outputs

- Cluster identifier assignment for every sequence in the input FASTA
- Per-dataset clustering metrics table (ACC, Homogeneity, Completeness, DBI, silhouette) for each method x dataset combination
- Aggregate accuracy-gap summary versus classical (k-means++, GMM) and specialized (MeShClust v3.0, DeLUCS) baselines

## How to apply

Feed a single FASTA file of DNA sequences plus an optional ground-truth TSV with 'sequence_id' and 'cluster_id' columns into iDeLUCS; ground-truth labels are never used during training, only for posthoc analysis. Choose k-mer length k from the supported options [4,5,6] (default 6), set --n_clusters to at least the true number of clusters (valid range 0 or 2-199; 0 triggers iDeLUCS + HDBSCAN for fine-grained clustering), keep the lambda=2.8 cluster-balance hyperparameter (1 = highly imbalanced, 3 = perfectly balanced), train for 50-150 epochs, and pick --batch_sz as a multiple of --n_mimics. Run the classical baselines (k-means++ and Gaussian Mixture Models on the same k-mer genomic signatures via scikit-learn) and the DNA-specialized baselines (DeLUCS and MeShClust v3.0, in both automatic and optimized identity-threshold modes) on identical inputs for fair comparison. Score every method x dataset cell with external metrics (unsupervised clustering accuracy via optimal matching of predicted IDs to ground truth, Homogeneity, Completeness) and intrinsic metrics (Davies-Bouldin index, silhouette), then aggregate per-family averages to quantify accuracy gaps between method classes.

## Related tools

- **iDeLUCS** (Primary tool: deep-learning detection of genomic signatures and unsupervised clustering of DNA sequences) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **PyTorch** (Machine learning development framework underlying iDeLUCS training)
- **DeLUCS** (DNA-specialized clustering baseline for accuracy comparison)
- **MeShClust v3.0** (DNA-specialized clustering baseline, run in automatic and optimized identity-threshold modes)
- **k-means++** (Classical baseline clustering on k-mer genomic signatures)
- **GMM** (Classical baseline (Gaussian Mixture Model) on k-mer genomic signatures)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Unsupervised clustering accuracy (ACC) computed by optimally matching predicted cluster IDs to ground-truth 'cluster_id' labels — never by direct label equality, since label permutation is arbitrary
- Reproduced aggregate gaps should match the reported finding: iDeLUCS beats the two classical algorithms by ~20% and the two specialized algorithms by ~12% on real DNA datasets
- Verify the GT file's sequence_id values correspond one-to-one with the FASTA sequence identifiers before posthoc scoring
- Per-dataset ACC values for iDeLUCS on the new mitochondrial datasets should fall in the 78%-89.7% range reported for Protista, Insecta, and Fungi
- Confirm ground-truth labels were used only post hoc, not during training

## Limitations

- No ACC benchmark results exist for synthetic datasets with average sequence length <500 bp or >200 clusters (e.g. 5000); the authors explicitly flag systematic testing of these regimes as future work
- k-mer length is restricted to k in [4,5,6]
- batch_sz must be a multiple of n_mimics and may be limited by machine capacity
- PyTorch's latest stable release might not be compatible with your version of CUDA; pip-installed PyQt5 does not support Apple Silicon (requires brew workaround)

## Evidence

- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [methods] iDeLUCS uses PyTorch as the machine learning development framework: "iDeLUCS uses PyTorch as the machine learning development framework"
- [readme] These labels won't be used during training, just for posthoc analysis: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id` with the sequence identifiers"
- [readme] Use 0 to launch iDeLCUS + HDBSCAn to automatically find the number of clusters in fine-grained clustering: "Use 0 to launch iDeLCUS + HDBSCAn to automatically find the number of clusters in fine-grained clustering.| Default = 5 ; Range: 0, 2-199."
