---
name: cli-hyperparameter-configuration
description: Use when invoking the idelucs CLI on a DNA FASTA dataset and you must
  choose valid hyperparameter values (k in [4,5,6], n_clusters 2-199, epochs 50-150,
  batch_sz a multiple of n_mimics).
when_to_use_negative:
- The dataset has more than 199 expected clusters (e.g., ~5000 clusters) — --n_clusters
  is restricted to the range 0, 2-199, and the authors flag >200-cluster datasets
  as future work
- Sequences are shorter than the regime the paper evaluated (short reads <500 bp were
  excluded from the published evaluation and flagged for future testing)
- The goal is alignment-based sequence analysis — iDeLUCS clusters genomic signatures
  without sequence alignment or taxonomic identifiers
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: The CLI tool whose hyperparameters are configured and run on FASTA DNA datasets
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: pip
  role: Installs the idelucs package from PyPI (pip install idelucs)
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training
- name: venv
  role: Optional virtual environment creation before installing iDeLUCS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:ab6d0d97d8965169e5432d1da816ee26c01cb3522b72de0ae343072d7150b7b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - PyTorch
  - Python
  - iDeLUCS
  - pip
  - venv
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# cli-hyperparameter-configuration

## Summary

Configure the iDeLUCS CLI hyperparameters (--k, --n_clusters, --n_epochs, --lambda, --n_mimics, --batch_sz) within the tool's documented supported and recommended ranges so that clustering runs on a given FASTA dataset are reproducible and within documented constraints.

## When to use

Use when invoking the idelucs CLI on a DNA FASTA dataset and you must choose valid hyperparameter values (k in [4,5,6], n_clusters 2-199, epochs 50-150, batch_sz a multiple of n_mimics).

## When NOT to use

- The dataset has more than 199 expected clusters (e.g., ~5000 clusters) — --n_clusters is restricted to the range 0, 2-199, and the authors flag >200-cluster datasets as future work
- Sequences are shorter than the regime the paper evaluated (short reads <500 bp were excluded from the published evaluation and flagged for future testing)
- The goal is alignment-based sequence analysis — iDeLUCS clusters genomic signatures without sequence alignment or taxonomic identifiers

## Inputs

- Single FASTA file of DNA sequences (path given via <sequence_file>)
- Optional tab-separated ground-truth file with columns sequence_id and cluster_id (--GT_file), used only for post hoc analysis
- User-selected hyperparameter values: --k, --n_clusters, --n_epochs, --n_mimics, --batch_sz, --lambda, --optimizer, --scheduler

## Outputs

- Cluster identifier assignments for every sequence in the input FASTA file
- Post hoc clustering metrics (e.g., ACC) when a --GT_file is provided

## How to apply

Install iDeLUCS via pip (`pip install idelucs`), then run the `idelucs` CLI with `--sequence_file` pointing to a single FASTA file and, when ground truth exists, `--GT_file` pointing to a tab-separated file with `sequence_id` and `cluster_id` columns used only for post hoc analysis. Set `--k` within the supported options [4,5,6] (default 6), `--n_clusters` to the expected cluster count (range 0, 2-199; use 0 to launch iDeLUCS + HDBSCAN for automatic cluster-number discovery), and `--n_epochs` within the recommended range [50,150]. Choose `--lambda` (default 2.8, range ~1 for imbalanced to 3 for perfectly balanced datasets) to control cluster balance, and pick `--batch_sz` as a multiple of `--n_mimics` so training pairs divide evenly across data augmentations. Verify the configuration by running `idelucs -h` first and confirming that `--n_clusters` is at least the true number of clusters whenever a GT file is supplied.

## Related tools

- **iDeLUCS** (The CLI tool whose hyperparameters are configured and run on FASTA DNA datasets) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **pip** (Installs the idelucs package from PyPI (pip install idelucs))
- **PyTorch** (Machine learning framework underlying iDeLUCS training)
- **venv** (Optional virtual environment creation before installing iDeLUCS)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Run `idelucs -h` and confirm the CLI parses the chosen arguments without error
- Check --k is one of [4,5,6] and --n_epochs lies within the recommended range [50,150]
- Check --n_clusters is within 0 or 2-199 and is >= n_true_clusters when a GT file is provided
- Check --batch_sz is a multiple of --n_mimics so training pairs divide evenly
- When a GT file is given, confirm post hoc ACC is computed against the cluster_id column

## Limitations

- --n_clusters supports only 0 and 2-199; the paper excluded datasets with very large expected cluster counts (e.g., 5000) and flagged testing >200 clusters as future work
- Short-read datasets (<500 bp average sequence length) were excluded from the published evaluation
- PyTorch's latest stable release might not be compatible with your CUDA version; batch_sz may also be limited by machine capacity
- The pip distribution of PyQt5 does not support Apple Silicon, requiring a homebrew workaround for the GUI

## Evidence

- [readme] k-mer length options: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [readme] n_clusters supported range: "--n_clusters| integer|Expected or maximum number of clusters to find. It should be at least `n_true_clusters` when `GT_file` is provided"
- [readme] GT file columns and post hoc use: "These labels won't be used during training, just for posthoc analysis."
- [methods] install via pip: [evidence span withheld — anchor status: not_found]
- [methods] PyTorch as ML framework: "iDeLUCS uses PyTorch as the machine learning development framework"
