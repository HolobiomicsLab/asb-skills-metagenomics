---
name: clustering-benchmark-metric-computation
description: Use when comparing clustering algorithms on labeled FASTA datasets with
  TSV ground-truth files, to produce a per-method × per-dataset table of intrinsic
  and external clustering evaluation metrics.
when_to_use_negative:
- The dataset has no ground-truth labels available — external metrics (Homogeneity,
  Completeness, ACC) cannot be computed, and iDeLUCS should be run without --GT_file
  for unknown datasets
- The task is supervised classification or sequence alignment-based clustering — iDeLUCS
  clusters 'without the need for sequence alignment or taxonomic identifiers', so
  this skill targets alignment-free unsupervised clustering evaluation
- You only need to run a single clustering method without benchmarking against baselines
  — use the method's own skill rather than this comparison workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3299
tools:
- name: iDeLUCS
  role: Deep-learning unsupervised clustering of DNA sequences via genomic signatures;
    PyTorch-based, run with k=6 and lambda=2.8
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-sequence clustering baseline used in the benchmark comparison
- name: MeShClust v3.0
  role: Specialized DNA-sequence clustering baseline, run in both automatic and optimized
    identity-threshold modes
- name: k-means++
  role: Classical clustering baseline for the benchmark
- name: GMM
  role: Gaussian Mixture Model, classical clustering baseline for the benchmark
- name: PyTorch
  role: Machine learning framework in which iDeLUCS clustering is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:b61cea4a481852aec4751d51d833443469b0a31f219ffdfb4ae0531cae61dcdd
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
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# clustering-benchmark-metric-computation

## Summary

Benchmark unsupervised DNA-sequence clustering methods (iDeLUCS, DeLUCS, MeShClust v3.0, k-means++, GMM) by computing intrinsic metrics (Davies-Bouldin index, silhouette) and external metrics (Homogeneity, Completeness, ACC) against ground-truth cluster_id labels, then assembling a Table 2-equivalent results table.

## When to use

Use when comparing clustering algorithms on labeled FASTA datasets with TSV ground-truth files, to produce a per-method × per-dataset table of intrinsic and external clustering evaluation metrics.

## When NOT to use

- The dataset has no ground-truth labels available — external metrics (Homogeneity, Completeness, ACC) cannot be computed, and iDeLUCS should be run without --GT_file for unknown datasets
- The task is supervised classification or sequence alignment-based clustering — iDeLUCS clusters 'without the need for sequence alignment or taxonomic identifiers', so this skill targets alignment-free unsupervised clustering evaluation
- You only need to run a single clustering method without benchmarking against baselines — use the method's own skill rather than this comparison workflow

## Inputs

- Mitochondrial genome FASTA files (e.g. Protista, Insecta, Fungi datasets)
- Ground-truth TSV files with columns 'sequence_id' and 'cluster_id' (optional 'phylogeny')
- True number of clusters per dataset (for --n_clusters)

## Outputs

- table2_clustering_metrics.csv with one row per method × dataset cell
- Per-method clustering assignments for each dataset
- Intrinsic metrics: Davies-Bouldin index, silhouette score
- External metrics: Homogeneity, Completeness, ACC

## How to apply

Load each mitochondrial genome FASTA dataset together with its ground-truth TSV containing the 'cluster_id' column (and optional 'phylogeny' column), where sequence identifiers in the GT file must match those in the FASTA file. Cluster each dataset with iDeLUCS in PyTorch using k-mer length 6 (options [4,5,6]) and lambda = 2.8 for cluster balance, setting --n_clusters to the true number of clusters and providing --GT_file for posthoc analysis only; run the classical baselines k-means++ and a Gaussian Mixture Model, plus the specialized algorithms DeLUCS and MeShClust v3.0 (MeShClust in both automatic and optimized identity-threshold modes). For every method × dataset combination, compute intrinsic metrics (Davies-Bouldin index, silhouette score) and external metrics (Homogeneity, Completeness, ACC) by posthoc comparison against the ground-truth cluster_id assignments. Assemble all values into a single Table 2-equivalent results table and validate that external metrics were computed against the ground-truth labels for all five methods on all three datasets.

## Related tools

- **iDeLUCS** (Deep-learning unsupervised clustering of DNA sequences via genomic signatures; PyTorch-based, run with k=6 and lambda=2.8) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-sequence clustering baseline used in the benchmark comparison)
- **MeShClust v3.0** (Specialized DNA-sequence clustering baseline, run in both automatic and optimized identity-threshold modes)
- **k-means++** (Classical clustering baseline for the benchmark)
- **GMM** (Gaussian Mixture Model, classical clustering baseline for the benchmark)
- **PyTorch** (Machine learning framework in which iDeLUCS clustering is executed)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- table2_clustering_metrics.csv exists and contains one row per method × dataset cell (5 methods × 3 datasets)
- External metrics (Homogeneity, Completeness, ACC) were computed against the ground-truth 'cluster_id' labels for all five methods on all three datasets
- iDeLUCS unsupervised clustering accuracies fall in the 78%–89.7% range reported for the new mitochondrial datasets
- iDeLUCS outperforms all other algorithms on the Insects and Fungi datasets (balanced and unbalanced) and performs comparably on the Protist dataset
- Sequence identifiers in the GT TSV correspond to identifiers in the FASTA sequence file

## Limitations

- iDeLUCS uses PyTorch as the ML framework and its latest stable release might not be compatible with your version of CUDA
- The GUI is built on Qt; pip's PyQt5 distribution does not support Apple Silicon, requiring a brew-installed PyQt5 symlink workaround
- GT labels are never used during training — they serve posthoc analysis only, so external metrics depend entirely on GT file quality and identifier matching
- batch_sz must be a multiple of n_mimics and may be limited by machine capacity

## Evidence

- [results] On the new mitochondrial datasets (b), iDeLUCS obtains unsupervised clustering accuracies ranging from 78% to 89.7%: [evidence span withheld — anchor status: not_found]
- [readme] GT labels posthoc only: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id` with the sequence identifiers"
- [readme] k-mer and lambda options: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [readme] lambda cluster balance: "--lambda| float | Hyperparameter to control cluster balance. | Default = 2.8; Recommended Range  1 (highly imbalanced dataset) - 3 (perfectly balanced dataset)"
- [readme] PyTorch framework: "iDeLUCS uses PyTorch as the machine learning development framework and its latest stable release might not be compatible with your version of CUDA"
