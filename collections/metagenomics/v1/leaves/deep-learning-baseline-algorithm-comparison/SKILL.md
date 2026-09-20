---
name: deep-learning-baseline-algorithm-comparison
description: Use when evaluating a deep unsupervised clustering tool for unlabeled
  DNA sequences (e.g. FASTA mitochondrial genomes) against k-means++, GMM, DeLUCS,
  and MeShClust v3.0 with ground-truth cluster_id TSV labels available for posthoc
  scoring.
when_to_use_negative:
- The sequences are already labeled or a supervised classifier is the goal — the ground-truth
  TSV must not be used during iDeLUCS training, only for posthoc comparison
- The input is a feature table or aligned sequences — iDeLUCS clusters raw DNA sequences
  via genomic signatures without alignment
- The number of clusters is unknown and fine-grained — although --n_clusters=0 launches
  iDeLUCS + HDBSCAN to infer cluster count automatically, this comparison workflow
  assumes the true cluster count is provided
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Deep-learning unsupervised clustering of DNA sequences in PyTorch (k=6, lambda=2.8);
    the method under evaluation
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: k-means++
  role: Classical baseline clustering algorithm compared against iDeLUCS
- name: GMM
  role: Classical Gaussian Mixture Model baseline compared against iDeLUCS
- name: DeLUCS
  role: DNA-specialized deep clustering baseline compared against iDeLUCS
- name: MeShClust v3.0
  role: DNA-specialized clustering baseline, run in both automatic and optimized identity-threshold
    modes
- name: PyTorch
  role: Machine learning framework used to implement and run iDeLUCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:c93936ef4f9b0a3fd150eb12b453e91715ec5256cb98f39e32c0e1a79b165581
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# deep-learning-baseline-algorithm-comparison

## Summary

Benchmark a deep-learning clustering method (iDeLUCS, PyTorch) against two classical baselines (k-means++, GMM) and two DNA-specialized algorithms (DeLUCS, MeShClust v3.0) on alignment-free DNA sequence clustering, scoring every method x dataset cell with intrinsic (DBI, silhouette) and external (Homogeneity, Completeness, ACC) metrics.

## When to use

Use when evaluating a deep unsupervised clustering tool for unlabeled DNA sequences (e.g. FASTA mitochondrial genomes) against k-means++, GMM, DeLUCS, and MeShClust v3.0 with ground-truth cluster_id TSV labels available for posthoc scoring.

## When NOT to use

- The sequences are already labeled or a supervised classifier is the goal — the ground-truth TSV must not be used during iDeLUCS training, only for posthoc comparison
- The input is a feature table or aligned sequences — iDeLUCS clusters raw DNA sequences via genomic signatures without alignment
- The number of clusters is unknown and fine-grained — although --n_clusters=0 launches iDeLUCS + HDBSCAN to infer cluster count automatically, this comparison workflow assumes the true cluster count is provided

## Inputs

- DNA sequence datasets in a single FASTA file per dataset (e.g. mitochondrial genomes from Protista, Insecta, Fungi)
- Ground-truth TSV file per dataset with 'sequence_id' and 'cluster_id' columns (optional 'phylogeny' column), used only for posthoc analysis
- True number of clusters per dataset (--n_clusters)

## Outputs

- Clustering metric table (CSV) with one row per method x dataset cell containing DBI, silhouette, Homogeneity, Completeness, and ACC
- Predicted cluster assignments per method for each dataset

## How to apply

Load each FASTA dataset (e.g. Protist, Insecta, Fungi mitochondrial genomes) with its ground-truth TSV containing 'sequence_id' and 'cluster_id' columns (the GT file is used for posthoc analysis only, never during training). Run iDeLUCS in PyTorch with k-mer length 6 (options [4,5,6]), lambda = 2.8 for cluster balance, --n_clusters set to the true number of clusters, and --GT_file supplied for posthoc evaluation. Cluster the same data with k-means++ and a Gaussian Mixture Model, plus the DNA-specialized DeLUCS and MeShClust v3.0, running MeShClust in both automatic and optimized identity-threshold modes. For every method x dataset combination, compute intrinsic metrics (Davies-Bouldin index, silhouette score) and external metrics (Homogeneity, Completeness, ACC) by comparing predicted clusters against the ground-truth 'cluster_id' assignments. Assemble all values into a single Table-2-equivalent CSV (e.g. table2_clustering_metrics.csv) with one row per method x dataset cell, then verify external metrics were computed against the 'cluster_id' labels for all five methods on all datasets.

## Related tools

- **iDeLUCS** (Deep-learning unsupervised clustering of DNA sequences in PyTorch (k=6, lambda=2.8); the method under evaluation) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **k-means++** (Classical baseline clustering algorithm compared against iDeLUCS)
- **GMM** (Classical Gaussian Mixture Model baseline compared against iDeLUCS)
- **DeLUCS** (DNA-specialized deep clustering baseline compared against iDeLUCS)
- **MeShClust v3.0** (DNA-specialized clustering baseline, run in both automatic and optimized identity-threshold modes)
- **PyTorch** (Machine learning framework used to implement and run iDeLUCS)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- table2_clustering_metrics.csv exists and contains exactly one row per method x dataset combination for all five methods on all three datasets
- External metrics (Homogeneity, Completeness, ACC) are present and were computed against the ground-truth 'cluster_id' labels for every cell
- Intrinsic metrics (Davies-Bouldin index, silhouette score) are reported for every method x dataset cell
- iDeLUCS unsupervised accuracies fall in the reported 78%-89.7% range on the new mitochondrial datasets, outperforming all baselines on Insecta and Fungi and performing comparably on Protista
- Baseline result reproduces the reported average advantage: iDeLUCS beats classical algorithms by ~20% and specialized algorithms by ~12% in unsupervised accuracy

## Limitations

- PyTorch's latest stable release might not be compatible with your version of CUDA, affecting reproducibility of iDeLUCS runs
- MeShClust v3.0 must be evaluated in both automatic and optimized identity-threshold modes; results can differ between the two modes
- batch_sz can be limited by machine capacity and a larger batch may improve convergence but may harm accuracy
- External metrics depend on GT files being provided for posthoc analysis only; iDeLUCS is not applicable when labels must be used during training

## Evidence

- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%"
- [readme] GT file usage: "These labels won't be used during training, just for posthoc analysis."
- [readme] k-mer length options: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [methods] PyTorch framework: "iDeLUCS uses PyTorch as the machine learning development framework"
- [readme] lambda balance hyperparameter: "--lambda| float | Hyperparameter to control cluster balance."
