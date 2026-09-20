---
name: benchmark-aggregate-gap-analysis
description: Use when a set of real DNA FASTA datasets with ground-truth cluster labels
  needs a head-to-head accuracy comparison between a deep-learning clustering method
  and classical or DNA-specialized baselines, with aggregate gap reporting.
when_to_use_negative:
- Datasets lack ground-truth labels, so post-hoc unsupervised clustering accuracy
  cannot be computed and gaps versus baselines cannot be scored
- Input is not raw DNA sequences (e.g. already a feature table or alignment), since
  iDeLUCS is alignment-free and operates on k-mer genomic signatures from FASTA files
- Only a single clustering algorithm is of interest without baseline comparison, since
  the skill's purpose is aggregate gap analysis across five methods
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Primary deep-learning clustering method whose accuracy is benchmarked; run
    via CLI with k=6 and lambda=2.8
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: k-means++
  role: Classical baseline clustering algorithm run on k-mer genomic signatures with
    scikit-learn
- name: GMM
  role: Classical baseline Gaussian Mixture Model fit on k-mer genomic signatures
    with scikit-learn
- name: MeShClust v3.0
  role: DNA-specialized baseline clustering algorithm run on the same real DNA datasets
- name: DeLUCS
  role: DNA-specialized baseline (predecessor of iDeLUCS) run on the same real DNA
    datasets
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:f52741ea3c0466714a7770b1cc56ab1e283e4b5febf50bef6032506860adeda4
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
yes_rate: 0.875
n_claims: 16
---

# benchmark-aggregate-gap-analysis

## Summary

Benchmarking skill for reproducing the aggregate unsupervised clustering accuracy comparison of iDeLUCS against classical (k-means++, GMM) and DNA-specialized (MeShClust v3.0, DeLUCS) baselines on 11 real DNA sequence datasets, quantifying the average accuracy gaps reported in the iDeLUCS paper.

## When to use

Use when a set of real DNA FASTA datasets with ground-truth cluster labels needs a head-to-head accuracy comparison between a deep-learning clustering method and classical or DNA-specialized baselines, with aggregate gap reporting.

## When NOT to use

- Datasets lack ground-truth labels, so post-hoc unsupervised clustering accuracy cannot be computed and gaps versus baselines cannot be scored
- Input is not raw DNA sequences (e.g. already a feature table or alignment), since iDeLUCS is alignment-free and operates on k-mer genomic signatures from FASTA files
- Only a single clustering algorithm is of interest without baseline comparison, since the skill's purpose is aggregate gap analysis across five methods

## Inputs

- Real DNA sequence FASTA files (e.g. Vertebrata.fas) from the 11 previously published datasets
- Ground-truth TSV label files with sequence_id and cluster_id columns
- scikit-learn k-mer genomic signature feature matrices for the classical baselines

## Outputs

- Per-dataset unsupervised clustering accuracy table for iDeLUCS, k-means++, GMM, MeShClust v3.0, and DeLUCS
- Aggregate gap summary written to benchmark_accuracy_results.csv (iDeLUCS vs classical ~20%, vs specialized ~12%)

## How to apply

Load the 11 real DNA FASTA datasets together with their TSV ground-truth files (sequence_id and cluster_id columns), keeping labels strictly for post-hoc scoring as iDeLUCS requires. Run iDeLUCS per dataset via its CLI with k-mer length k=6, n_clusters set to at least the number of true clusters, and the default lambda=2.8 balance hyperparameter. Fit k-means++ and Gaussian Mixture Models on the k-mer genomic signatures with scikit-learn as classical baselines, and run the DNA-specialized baselines DeLUCS and MeShClust v3.0 on the same data. Score all five algorithms with unsupervised clustering accuracy by optimally matching predicted cluster IDs to ground-truth labels, then aggregate per-family averages across vertebrate mtDNA, Bacteria, and viral datasets and compute iDeLUCS's accuracy gaps versus classical and specialized methods, writing the per-dataset table and gap summary to benchmark_accuracy_results.csv.

## Related tools

- **iDeLUCS** (Primary deep-learning clustering method whose accuracy is benchmarked; run via CLI with k=6 and lambda=2.8) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **k-means++** (Classical baseline clustering algorithm run on k-mer genomic signatures with scikit-learn)
- **GMM** (Classical baseline Gaussian Mixture Model fit on k-mer genomic signatures with scikit-learn)
- **MeShClust v3.0** (DNA-specialized baseline clustering algorithm run on the same real DNA datasets)
- **DeLUCS** (DNA-specialized baseline (predecessor of iDeLUCS) run on the same real DNA datasets)
- **PyTorch** (Machine learning framework underlying iDeLUCS training)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5
```

## Evaluation signals

- Reproduced aggregate gap matches the reported finding: iDeLUCS outperforms the two classical algorithms (k-means++, GMM) by an average of ~20% and the two specialized algorithms (MeShClust v3.0, DeLUCS) by an average of ~12%
- Per-dataset accuracy table covers all 11 real datasets and all five algorithms, with predicted cluster IDs optimally matched to ground-truth cluster_id labels before scoring
- iDeLUCS runs used the paper's hyperparameters: k=6, lambda=2.8, and n_clusters at least equal to the number of true clusters in each GT file
- Output file benchmark_accuracy_results.csv exists and contains both the per-dataset accuracy table and the aggregate gap summary
- Ground-truth labels were used only for post-hoc scoring, not during iDeLUCS training, per the tool's design

## Limitations

- iDeLUCS's latest stable PyTorch release may not be compatible with the user's CUDA version
- Aggregate gaps are averages over the specific 11 real datasets analyzed (vertebrate mtDNA, Bacteria, viral), not a universal guarantee
- Unsupervised clustering accuracy depends on optimal matching of predicted cluster IDs to ground truth; batch size must be a multiple of n_mimics and larger batches may harm accuracy

## Evidence

- [abstract] iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [readme] GT file labels used only post-hoc: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id` with the sequence identifiers"
- [readme] Example CLI invocation with k=6 and lambda=2.8: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512"
- [readme] n_clusters must cover true clusters: "It should be at least `n_true_clusters` when `GT_file` is provided'."
