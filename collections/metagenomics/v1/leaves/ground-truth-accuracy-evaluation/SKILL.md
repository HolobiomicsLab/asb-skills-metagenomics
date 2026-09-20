---
name: ground-truth-accuracy-evaluation
description: Use when you need to quantify clustering accuracy (ACC) of iDeLUCS output
  against known cluster labels, e.g. reproducing the 98.5% (given k) or 97.3% (HDBSCAN
  auto-k) synthetic-dataset results or benchmarking excluded regimes.
when_to_use_negative:
- 'The dataset has no trusted labels: --GT_file is only for post hoc scoring, so without
  a ground-truth cluster_id column no ACC can be computed'
- The dataset falls outside the evaluated regimes — average sequence length below
  500 bp or more than 200 clusters (e.g. ~5000) — since the paper excluded these and
  reports no ACC results for them
- 'You need true labels to influence training: the workflow forbids using GT labels
  during training, so this skill cannot be used as a supervised method'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Deep-learning clustering tool whose predictions are scored against the ground-truth
    cluster_id labels via the --GT_file mechanism
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: HDBSCAN
  role: Used when --n_clusters=0 to automatically determine the number of clusters
    during fine-grained clustering, enabling the auto-k accuracy evaluation
- name: pip
  role: Installs the idelucs CLI package used to run clustering with GT-file-based
    accuracy evaluation
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training and inference
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:bf2cc3e8345e7ecd57f1768e52c183d9f80708f3b42ffbfaf9bd891364986793
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - HDBSCAN
  - MeShClust v3.0
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ground-truth-accuracy-evaluation

## Summary

Evaluates iDeLUCS DNA-sequence clustering quality by supplying a tab-separated ground-truth file (sequence_id, cluster_id) that is never used during training, only for post hoc computation of unsupervised clustering accuracy (ACC).

## When to use

Use when you need to quantify clustering accuracy (ACC) of iDeLUCS output against known cluster labels, e.g. reproducing the 98.5% (given k) or 97.3% (HDBSCAN auto-k) synthetic-dataset results or benchmarking excluded regimes.

## When NOT to use

- The dataset has no trusted labels: --GT_file is only for post hoc scoring, so without a ground-truth cluster_id column no ACC can be computed
- The dataset falls outside the evaluated regimes — average sequence length below 500 bp or more than 200 clusters (e.g. ~5000) — since the paper excluded these and reports no ACC results for them
- You need true labels to influence training: the workflow forbids using GT labels during training, so this skill cannot be used as a supervised method

## Inputs

- Single FASTA file of DNA training sequences (--sequence_file)
- Tab-separated ground-truth file with `sequence_id` and `cluster_id` columns (optional `phylogeny` column), supplied via --GT_file
- Known or expected number of clusters (for fixed-cluster mode; 0 for HDBSCAN auto-k mode)

## Outputs

- Per-sequence cluster identifier assignments for every sequence in the FASTA input
- Unsupervised clustering accuracy (ACC) values per dataset, aggregatable into mode-wise averages (e.g. TSV/CSV benchmark table)

## How to apply

Prepare a TSV ground-truth file with a `sequence_id` column whose identifiers match those in the input FASTA, and a `cluster_id` column holding the true cluster assignments; pass it to the idelucs CLI via --GT_file. The labels are explicitly not used during training — only for post hoc analysis — so ACC is computed by comparing predicted cluster identifiers to cluster_id after clustering finishes. Run iDeLUCS in fixed-cluster mode (--n_clusters set to the known count) and, optionally, in automatic mode with --n_clusters=0, which launches iDeLUCS + HDBSCAN to determine the number of clusters. Keep hyperparameters within documented ranges: k in [4,5,6] (default 6), lambda=2.8, n_epochs 50-150, and --batch_sz a multiple of --n_mimics. Note the documented limits: --n_clusters range is 0 and 2-199, and the evaluation excluded short reads (<500 bp) and very large cluster counts (>200, e.g. ~5000), which the authors flag as future work.

## Related tools

- **iDeLUCS** (Deep-learning clustering tool whose predictions are scored against the ground-truth cluster_id labels via the --GT_file mechanism) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **HDBSCAN** (Used when --n_clusters=0 to automatically determine the number of clusters during fine-grained clustering, enabling the auto-k accuracy evaluation)
- **pip** (Installs the idelucs CLI package used to run clustering with GT-file-based accuracy evaluation)
- **PyTorch** (Machine learning framework underlying iDeLUCS training and inference)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- The output benchmark table contains an ACC value for every dataset, computed against the ground-truth cluster_id column used post hoc
- Reproduced mode averages are consistent with the paper: 98.5% ACC with the number of clusters given and 97.3% with HDBSCAN auto-k, alongside MeShClust v3.0's 99.3% reference
- GT file identifiers in `sequence_id` correspond exactly to identifiers in the FASTA sequence file, and cluster_id values match the expected n_true_clusters
- --n_clusters used is within the documented range (0, 2-199); k is within [4,5,6]; --batch_sz is a multiple of --n_mimics

## Limitations

- Datasets with average sequence length < 500 bp or more than 200 clusters (~5000 in the discussed regime) were excluded from the paper's evaluation; systematic testing there is explicitly flagged as future work, so no ACC benchmarks exist
- --n_clusters is capped at 199 (range 0, 2-199), so very large cluster counts cannot be requested
- Ground-truth labels are never used during training, only for post hoc analysis — this evaluation cannot improve clustering itself
- PyTorch's latest stable release may not be compatible with the local CUDA version, and --batch_sz may be limited by machine capacity

## Evidence

- [readme] Ground-truth labels post hoc: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] HDBSCAN auto-k mode: "Use 0 to launch iDeLCUS + HDBSCAn to automatically find the number of clusters in fine-grained clustering."
- [results] Auto-k accuracy result: [evidence span withheld — anchor status: not_found]
- [discussion] Excluded regimes / future work: [evidence span withheld — anchor status: not_found]
- [readme] GT example invocation: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6"
