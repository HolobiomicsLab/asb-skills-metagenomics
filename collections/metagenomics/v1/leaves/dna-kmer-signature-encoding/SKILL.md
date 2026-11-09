---
name: dna-kmer-signature-encoding
description: Use when raw DNA reads or sequences in FASTA must be converted to numeric
  signatures for unsupervised alignment-free clustering (e.g. iDeLUCS, DeLUCS, k-means++),
  with ground truth reserved for post-hoc scoring.
when_to_use_negative:
- The analysis requires full sequence alignment, assembly, or phylogenetic placement
  — this skill is explicitly alignment-free and taxonomy-identifier-free signature
  clustering
- The number of true clusters is unknown but you cannot tolerate HDBSCAN's automatic
  estimation — n_clusters=0 mode trades accuracy for auto-k (97.3% vs 98.5% ACC on
  the Girgis 2022 synthetic datasets)
- Input is already a numeric feature/embedding table rather than raw DNA FASTA sequences
  — no re-encoding is needed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Consumes the k-mer-encoded sequences and performs unsupervised deep learning
    clustering via contrastive loss with mimic-based data augmentation
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-clustering baseline run on the same k-mer-encoded sequences
    with matched hyperparameters
- name: k-means++
  role: Classical scikit-learn baseline (e.g. k=8) applied to the identical signature
    vectors for benchmark comparison
- name: HDBSCAN
  role: Automatically determines the number of clusters when iDeLUCS is launched with
    n_clusters=0
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:191ff0a540233545c3fefed0f5284132cfe8e2000a55b42c9eb1b7b373b99bd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - HDBSCAN
  - MeShClust v3.0
  - PyTorch
  - Python
  - iDeLUCS
  - k-means++
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# dna-kmer-signature-encoding

## Summary

Encode DNA sequences as fixed-length k-mer frequency signature vectors (k between 4 and 6, default k=6) so alignment-free deep learning clustering tools like iDeLUCS can detect genomic signatures and cluster sequences without alignment or taxonomic identifiers.

## When to use

Use when raw DNA reads or sequences in FASTA must be converted to numeric signatures for unsupervised alignment-free clustering (e.g. iDeLUCS, DeLUCS, k-means++), with ground truth reserved for post-hoc scoring.

## When NOT to use

- The analysis requires full sequence alignment, assembly, or phylogenetic placement — this skill is explicitly alignment-free and taxonomy-identifier-free signature clustering
- The number of true clusters is unknown but you cannot tolerate HDBSCAN's automatic estimation — n_clusters=0 mode trades accuracy for auto-k (97.3% vs 98.5% ACC on the Girgis 2022 synthetic datasets)
- Input is already a numeric feature/embedding table rather than raw DNA FASTA sequences — no re-encoding is needed

## Inputs

- Single FASTA file of DNA sequences or reads (e.g. PacBio-simulated metagenomic reads, 432,333 reads from 8 microbial genomes)
- Optional ground-truth TSV with sequence_id and cluster_id columns (optionally phylogeny) whose identifiers match the FASTA
- Hyperparameters: k in [4,5,6] (default 6), n_clusters (2-199 or 0 for HDBSCAN auto-k), n_epochs, n_mimics, batch_sz, lambda, optimizer, scheduler

## Outputs

- k-mer frequency signature vectors (one per sequence, fixed length determined by k)
- Per-sequence cluster identifier assignments from iDeLUCS (or iDeLUCS + HDBSCAN when n_clusters=0)
- Post-hoc unsupervised clustering accuracy (ACC) tables comparing methods, e.g. accuracy_comparison.csv

## How to apply

Load a single FASTA file of DNA sequences and compute k-mer frequency signatures with k selected from the allowed options [4, 5, 6]; k=6 is the default used in both the iDeLUCS README examples (Vertebrata and Actinopterygii datasets) and the simulated metagenomic reads benchmark. The k-mer choice determines the signature length and must stay consistent across methods being compared — e.g. DeLUCS and scikit-learn k-means++ (k=8 clusters) must run on the identical signature vectors. Feed the encoded sequences to iDeLUCS (PyTorch-backed, contrastive learning with mimic-based data augmentation) using n_epochs in the recommended 50-150 range, n_mimics 50-150, batch_sz a multiple of n_mimics, and lambda between 1 (highly imbalanced data) and 3 (perfectly balanced data). If ground-truth labels exist, pass them only via a --GT_file TSV with sequence_id and cluster_id columns for post-hoc analysis; they are never used during training. When the true number of clusters is unknown, set n_clusters=0 to launch iDeLUCS + HDBSCAN automatic cluster-number detection.

## Related tools

- **iDeLUCS** (Consumes the k-mer-encoded sequences and performs unsupervised deep learning clustering via contrastive loss with mimic-based data augmentation) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-clustering baseline run on the same k-mer-encoded sequences with matched hyperparameters)
- **k-means++** (Classical scikit-learn baseline (e.g. k=8) applied to the identical signature vectors for benchmark comparison)
- **HDBSCAN** (Automatically determines the number of clusters when iDeLUCS is launched with n_clusters=0)
- **PyTorch** (Machine learning framework underlying iDeLUCS training)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- On the simulated metagenomic reads benchmark (432,333 reads, 8 genomes), iDeLUCS on k=6 signatures should exceed DeLUCS by ~16% and k-means++ by ~7% in unsupervised clustering accuracy
- On the 12 synthetic datasets from Girgis (2022), average ACC should be ~98.5% with a given number of clusters and ~97.3% with HDBSCAN auto-k, vs MeShClust v3.0's reported 99.3%
- n_clusters must be at least the number of true clusters when a GT_file is provided; batch_sz must be a multiple of n_mimics
- Ground-truth cluster_id labels appear only in post-hoc optimal label alignment, never during training
- Accuracy comparison table (e.g. accuracy_comparison.csv) contains one accuracy row per method with measured pairwise gaps consistent with reported values

## Limitations

- k-mer length is restricted to options [4, 5, 6], with k=6 as the default; other k values are not supported
- n_clusters range is 0 and 2-199, so datasets needing more than 199 clusters are out of scope
- batch_sz may be limited by machine memory/capacity; a larger batch may improve convergence but harm accuracy
- The latest stable PyTorch release may not be compatible with your CUDA version; GUI users need Qt platform plugins and PyQt5 does not support Apple Silicon natively (a brew-based workaround is documented)

## Evidence

- [readme] k-mer length options: "--k| integer|k-mer length|  Default = 6, Options:`[4,5,6]`"
- [readme] GT file purpose: "These labels won't be used during training, just for posthoc analysis. The GT file must contain the columns: `sequence_id`"
- [readme] HDBSCAN auto-k mode: "Use 0 to launch iDeLCUS + HDBSCAn to automatically find the number of clusters in fine-grained clustering."
- [abstract] alignment-free signature clustering claim: "that detects genomic signatures and uses them to cluster DNA sequences, without the need for sequence alignment or taxonomic identifiers"
- [methods] PyTorch dependency: "iDeLUCS uses PyTorch as the machine learning development framework"
