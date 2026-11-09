---
name: ground-truth-posthoc-label-evaluation
description: Use when you have clustered unlabeled DNA sequences from a FASTA file
  and possess a ground-truth TSV with 'cluster_id' labels that must be used only for
  posthoc scoring of intrinsic and external clustering metrics.
when_to_use_negative:
- The dataset has no ground-truth cluster_id labels available — only intrinsic metrics
  can be computed and the GT_file workflow does not apply
- You intend to use the labels to supervise or tune the clustering itself, which would
  violate the posthoc-only design of the --GT_file argument
- Input is not raw DNA sequence clustering output but, e.g., an already-computed feature
  table or taxonomy profile
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Unsupervised deep-learning clusterer whose predictions are scored posthoc
    against --GT_file labels using external and intrinsic metrics
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA clustering baseline whose assignments are evaluated against
    the same ground-truth cluster_id labels
- name: MeShClust v3.0
  role: Specialized DNA clustering baseline run in automatic and optimized identity-threshold
    modes, scored against ground truth
- name: k-means++
  role: Classical clustering baseline evaluated against ground-truth cluster_id labels
- name: GMM
  role: Gaussian Mixture Model baseline evaluated against ground-truth cluster_id
    labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:24d1147cda3768b231c4ee91ad347e5c47d15329e61524b56477b7905586781c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Ground-truth posthoc label evaluation for DNA sequence clustering

## Summary

Evaluates unsupervised DNA clustering results (e.g. from iDeLUCS, DeLUCS, MeShClust v3.0, k-means++, GMM) by posthoc comparison of predicted cluster assignments against ground-truth 'cluster_id' labels, computing intrinsic (Davies-Bouldin, silhouette) and external (Homogeneity, Completeness, ACC) metrics without leaking labels into training.

## When to use

Use when you have clustered unlabeled DNA sequences from a FASTA file and possess a ground-truth TSV with 'cluster_id' labels that must be used only for posthoc scoring of intrinsic and external clustering metrics.

## When NOT to use

- The dataset has no ground-truth cluster_id labels available — only intrinsic metrics can be computed and the GT_file workflow does not apply
- You intend to use the labels to supervise or tune the clustering itself, which would violate the posthoc-only design of the --GT_file argument
- Input is not raw DNA sequence clustering output but, e.g., an already-computed feature table or taxonomy profile

## Inputs

- FASTA file of DNA sequences (e.g. mitochondrial genome datasets Protista, Insecta, Fungi)
- Ground-truth TSV file with 'sequence_id' and 'cluster_id' columns (optional 'phylogeny' column)
- Predicted cluster assignments from each clustering method (iDeLUCS, DeLUCS, MeShClust v3.0, k-means++, GMM)

## Outputs

- External metric values (Homogeneity, Completeness, ACC) per method x dataset combination
- Intrinsic metric values (Davies-Bouldin index, silhouette score) per method x dataset combination
- Consolidated benchmark results table (e.g. table2_clustering_metrics.csv)

## How to apply

Provide the ground-truth file via the --GT_file argument alongside the FASTA sequence file; per the README these labels 'won't be used during training, just for posthoc analysis', so the clustering itself must remain unsupervised (e.g. iDeLUCS with k=6, --n_clusters equal to the true number of clusters, lambda=2.8). The GT TSV must contain a 'sequence_id' column matching the FASTA identifiers and a 'cluster_id' column, optionally a 'phylogeny' column for alternative labeling. After clustering each method x dataset cell, compute intrinsic metrics (Davies-Bouldin index, silhouette score) on the learned embeddings/assignments and external metrics (Homogeneity, Completeness, ACC) by aligning predicted clusters to the ground-truth cluster_id assignments. In the iDeLUCS Table 2 benchmark on Protista, Insecta, and Fungi mitochondrial genomes, this protocol yields unsupervised accuracies of 78%–89.7% for iDeLUCS against k-means++, GMM, DeLUCS, and MeShClust v3.0 baselines.

## Related tools

- **iDeLUCS** (Unsupervised deep-learning clusterer whose predictions are scored posthoc against --GT_file labels using external and intrinsic metrics) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA clustering baseline whose assignments are evaluated against the same ground-truth cluster_id labels)
- **MeShClust v3.0** (Specialized DNA clustering baseline run in automatic and optimized identity-threshold modes, scored against ground truth)
- **k-means++** (Classical clustering baseline evaluated against ground-truth cluster_id labels)
- **GMM** (Gaussian Mixture Model baseline evaluated against ground-truth cluster_id labels)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- The results file contains one row per method x dataset cell (5 methods x 3 mitochondrial datasets for Table 2)
- External metrics (Homogeneity, Completeness, ACC) were computed against the ground-truth 'cluster_id' labels, never used during training
- GT TSV 'sequence_id' values match the identifiers in the FASTA sequence file exactly
- iDeLUCS external accuracy on the new mitochondrial datasets falls in the reported 78%–89.7% unsupervised range
- --n_clusters was set to the true number of clusters for each dataset so cluster counts are comparable across methods

## Limitations

- GT_file labels are not used during training, only for posthoc analysis, so they cannot improve the clustering itself
- PyTorch latest stable release might not be compatible with the user's version of CUDA
- --n_clusters must be at least n_true_clusters when a GT_file is provided; too few clusters invalidates the comparison

## Evidence

- [readme] GT_file usage restricted to posthoc analysis: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] cluster_id ground truth column requirement: "The column `cluster_id` with the "ground truth" assignments for each sequence."
- [abstract] iDeLUCS outperforms baselines on real DNA datasets: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%"
- [readme] n_clusters constraint with ground truth: "It should be at least `n_true_clusters` when `GT_file` is provided"
- [results] Table 2 finding on new mitochondrial datasets: [evidence span withheld — anchor status: not_found]
