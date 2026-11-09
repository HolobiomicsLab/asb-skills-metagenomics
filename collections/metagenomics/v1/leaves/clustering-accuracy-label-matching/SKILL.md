---
name: clustering-accuracy-label-matching
description: Use when comparing cluster assignments from unsupervised methods (e.g.,
  iDeLUCS, k-means++, GMM, DeLUCS, MeShClust v3.0) against ground-truth TSV labels,
  since predicted cluster IDs need optimal matching before accuracy can be aggregated.
when_to_use_negative:
- Ground-truth labels are needed as training input — iDeLUCS and this scoring protocol
  require labels to be withheld until posthoc analysis
- The number of predicted clusters differs from n_true_clusters and no optimal label-matching
  permutation is computed, making raw cluster-ID equality comparisons meaningless
- Sequences require alignment-based clustering rather than alignment-free genomic-signature
  clustering (k-mer signatures, k=4-6)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Deep-learning unsupervised clustering tool whose predictions are scored against
    ground truth with matched unsupervised clustering accuracy
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: DNA-specialized clustering baseline scored with the same matched-accuracy
    metric
- name: MeShClust v3.0
  role: DNA-specialized clustering baseline scored with the same matched-accuracy
    metric
- name: k-means++
  role: Classical baseline run on k-mer genomic signatures with scikit-learn and scored
    with matched accuracy
- name: GMM
  role: Gaussian Mixture Model classical baseline fit on k-mer genomic signatures
    with scikit-learn and scored with matched accuracy
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS used to produce cluster predictions
    that are subsequently scored
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:cc224bf0c49b433d26bfa4c3090cba8b18bd208e68fe5bcb1c7e0c01fa31d522
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# clustering-accuracy-label-matching

## Summary

Scores unsupervised clustering results on DNA sequence datasets by optimally matching predicted cluster IDs to ground-truth labels and computing unsupervised clustering accuracy, as used to benchmark iDeLUCS against k-means++, GMM, MeShClust v3.0, and DeLUCS on 11 real DNA datasets.

## When to use

Use when comparing cluster assignments from unsupervised methods (e.g., iDeLUCS, k-means++, GMM, DeLUCS, MeShClust v3.0) against ground-truth TSV labels, since predicted cluster IDs need optimal matching before accuracy can be aggregated.

## When NOT to use

- Ground-truth labels are needed as training input — iDeLUCS and this scoring protocol require labels to be withheld until posthoc analysis
- The number of predicted clusters differs from n_true_clusters and no optimal label-matching permutation is computed, making raw cluster-ID equality comparisons meaningless
- Sequences require alignment-based clustering rather than alignment-free genomic-signature clustering (k-mer signatures, k=4-6)

## Inputs

- FASTA files of real DNA sequences (e.g., Example/Vertebrata.fas) for each of the 11 published datasets
- Ground-truth TSV label files with sequence_id and cluster_id columns (optional phylogeny column)
- Predicted cluster assignments from each algorithm: iDeLUCS, k-means++ (scikit-learn), GMM (scikit-learn), DeLUCS, MeShClust v3.0

## Outputs

- Per-dataset unsupervised clustering accuracy table for all five algorithms
- Aggregate per-family averages across vertebrate mtDNA, Bacteria, and viral datasets
- benchmark_accuracy_results.csv with per-dataset accuracies and aggregate gap summary (iDeLUCS vs classical ~20%, vs specialized ~12%)

## How to apply

Load each FASTA dataset with its ground-truth TSV file containing sequence_id and cluster_id columns, keeping labels strictly for posthoc scoring as required by iDeLUCS (they are never used during training). After running each clustering algorithm, optimally match the predicted cluster identifiers to the ground-truth cluster_id assignments (e.g., via a permutation that maximizes agreement) before computing unsupervised clustering accuracy per dataset. Score all algorithms on every dataset with the same matched-accuracy metric so results are comparable across classical and DNA-specialized baselines. Aggregate per-family averages across the vertebrate mtDNA, Bacteria, and viral dataset groups and compute accuracy gaps, then write the per-dataset accuracy table and aggregate gap summary to benchmark_accuracy_results.csv. Validation: reproduced gaps should match the reported finding that iDeLUCS outperforms the classical algorithms (k-means++ and GMM) by ~20% and the specialized algorithms (MeShClust v3.0 and DeLUCS) by ~12% on average.

## Related tools

- **iDeLUCS** (Deep-learning unsupervised clustering tool whose predictions are scored against ground truth with matched unsupervised clustering accuracy) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (DNA-specialized clustering baseline scored with the same matched-accuracy metric)
- **MeShClust v3.0** (DNA-specialized clustering baseline scored with the same matched-accuracy metric)
- **k-means++** (Classical baseline run on k-mer genomic signatures with scikit-learn and scored with matched accuracy)
- **GMM** (Gaussian Mixture Model classical baseline fit on k-mer genomic signatures with scikit-learn and scored with matched accuracy)
- **PyTorch** (Machine learning framework underlying iDeLUCS used to produce cluster predictions that are subsequently scored)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- Reproduced aggregate gaps match the finding: iDeLUCS outperforms k-means++ and GMM by ~20% and MeShClust v3.0 and DeLUCS by ~12% on average
- Predicted cluster IDs are optimally permuted to ground-truth labels before accuracy is computed (no raw-ID comparison)
- GT files contain sequence_id identifiers matching those in the FASTA sequence_file, plus a cluster_id column
- Per-family averages are aggregated across vertebrate mtDNA, Bacteria, and viral dataset groups
- benchmark_accuracy_results.csv contains both the per-dataset accuracy table and the aggregate gap summary

## Limitations

- Ground-truth TSV labels are used only for posthoc scoring, not during training; misalignment between sequence_id values in FASTA and TSV breaks scoring
- n_clusters should be set at least to n_true_clusters when a GT_file is provided, otherwise matching-based accuracy is ill-defined
- iDeLUCS PyTorch version compatibility depends on the local CUDA version, which can affect reproducibility of scored results

## Evidence

- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [readme] GT_file labels are posthoc only: "These labels won't be used during training, just for posthoc analysis."
- [readme] GT file column requirements: [evidence span withheld — anchor status: not_found]
- [readme] n_clusters constraint with ground truth: "It should be at least `n_true_clusters` when `GT_file` is provided"
- [abstract] clustering without alignment: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
