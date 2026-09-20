---
name: synthetic-benchmark-reproduction
description: Use when a paper reports average unsupervised clustering accuracy on
  a specific benchmark of synthetic FASTA datasets and the numbers must be independently
  reproduced from the original datasets and tool parameters.
when_to_use_negative:
- The task is de novo clustering of your own unlabeled sequences rather than reproduction
  of a published benchmark with ground truth
- The benchmark datasets or ground-truth labels cannot be obtained, since ACC cannot
  be computed without cluster_id labels
- Sequences require alignment-based comparison — iDeLUCS is explicitly alignment-free
  and clusters via genomic signatures
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: iDeLUCS
  role: Deep learning CLI that clusters DNA sequences via k-mer genomic signatures;
    run in fixed-cluster mode (--n_clusters=k) and auto mode (--n_clusters=0) with
    --GT_file for post-hoc scoring
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: HDBSCAN
  role: Automatic cluster-number determination invoked inside iDeLUCS when n_clusters=0
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training
- name: MeShClust v3.0
  role: Published reference method whose 99.3% average accuracy on the same synthetic
    benchmark is the comparison target
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:560111d674c181583bb3a98716883b42246bfe8b3413897157aec5d89c2c713c
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# synthetic-benchmark-reproduction

## Summary

Reproduce published clustering-accuracy benchmarks (e.g. iDeLUCS's 98.5% ACC with given k and 97.3% with HDBSCAN auto-k on twelve synthetic datasets from Girgis 2022) by running the tool in both fixed-cluster and automatic modes and comparing against reference methods such as MeShClust v3.0 (99.3%).

## When to use

Use when a paper reports average unsupervised clustering accuracy on a specific benchmark of synthetic FASTA datasets and the numbers must be independently reproduced from the original datasets and tool parameters.

## When NOT to use

- The task is de novo clustering of your own unlabeled sequences rather than reproduction of a published benchmark with ground truth
- The benchmark datasets or ground-truth labels cannot be obtained, since ACC cannot be computed without cluster_id labels
- Sequences require alignment-based comparison — iDeLUCS is explicitly alignment-free and clusters via genomic signatures

## Inputs

- Twelve synthetic DNA sequence datasets in FASTA format from Girgis (2022)
- Ground-truth TSV files with `sequence_id` and `cluster_id` columns matching FASTA identifiers
- Published reference accuracy (MeShClust v3.0: 99.3% on the same benchmark)

## Outputs

- Per-dataset unsupervised clustering accuracy (ACC) for fixed-cluster mode (target average 98.5%)
- Per-dataset ACC for HDBSCAN auto-k mode (target average 97.3%)
- A single CSV table with per-dataset and averaged ACC for both modes plus the MeShClust v3.0 99.3% reference

## How to apply

Collect the benchmark FASTA datasets from the original source (Girgis 2022, 12 synthetic datasets) and build ground-truth TSV files with `sequence_id` and `cluster_id` columns matching the FASTA identifiers. Run iDeLUCS in fixed-cluster mode with the GT file supplied for post-hoc scoring only (e.g. k=6, lambda=2.8, n_clusters set to the known number of clusters, GT labels never used during training). Re-run the same datasets in automatic mode by setting n_clusters=0, which launches iDeLUCS + HDBSCAN to infer the cluster count. Compute unsupervised clustering accuracy (ACC) per dataset by matching predicted cluster_id against ground truth post hoc, aggregate into mode-wise averages, and tabulate alongside the MeShClust v3.0 reference accuracy of 99.3% in a single CSV.

## Related tools

- **iDeLUCS** (Deep learning CLI that clusters DNA sequences via k-mer genomic signatures; run in fixed-cluster mode (--n_clusters=k) and auto mode (--n_clusters=0) with --GT_file for post-hoc scoring) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **HDBSCAN** (Automatic cluster-number determination invoked inside iDeLUCS when n_clusters=0)
- **PyTorch** (Machine learning framework underlying iDeLUCS training)
- **MeShClust v3.0** (Published reference method whose 99.3% average accuracy on the same synthetic benchmark is the comparison target)

## Examples

```
idelucs --sequence_file=synthetic_01.fas --GT_file=synthetic_01_GT.tsv --n_epochs=50 --lambda=2.8 --k=6 --n_clusters=<known_k> --n_mimics=50 --batch_sz=256
```

## Evaluation signals

- GT TSV columns are exactly `sequence_id` and `cluster_id` and every sequence_id matches an identifier in the FASTA file
- Reproduced fixed-cluster mode average ACC is consistent with the reported 98.5% across the 12 synthetic datasets
- Reproduced HDBSCAN auto-k mode average ACC is consistent with the reported 97.3%
- Validation table contains per-dataset ACC values for both modes for all 12 datasets alongside the 99.3% MeShClust v3.0 reference
- GT labels are never used during training — only for post-hoc ACC computation (unsupervised protocol preserved)

## Limitations

- k-mer length is restricted to [4,5,6] (default k=6), so no other k values can be benchmarked
- iDeLUCS relies on PyTorch, whose latest stable release may not be compatible with your CUDA version
- batch_sz must be a multiple of n_mimics and may be limited by machine memory; larger batches may harm accuracy
- n_clusters should be at least n_true_clusters when a GT_file is provided

## Evidence

- [methods] Synthetic ACC with given k vs auto-k: [evidence span withheld — anchor status: not_found]
- [readme] Auto-k mechanism: "Use 0 to launch iDeLCUS + HDBSCAn to automatically find the number of clusters in fine-grained clustering."
- [readme] GT file format and usage: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] Representative CLI invocation: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6"
- [abstract] Alignment-free clustering premise: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
