---
name: unsupervised-clustering-accuracy-evaluation
description: Use when you have predicted cluster assignments for DNA sequences and
  a ground-truth TSV with sequence_id and cluster_id columns, and need to quantify
  clustering quality without training on labels.
when_to_use_negative:
- No ground-truth labels exist for the dataset — the GT_file requires cluster_id assignments;
  without them ACC cannot be computed.
- You need labels during training — iDeLUCS is unsupervised; the GT labels 'won't
  be used during training, just for posthoc analysis', so this skill must never influence
  clustering itself.
- Input is not DNA sequence clustering output (e.g., already a feature table or metadata
  matrix) — the skill evaluates cluster identifier assignments against biological
  truth labels.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: iDeLUCS
  role: Produces the cluster assignments being evaluated; PyTorch-backed deep learning
    clustering with contrastive loss and information-theoretic ensemble
  repo: https://github.com/Kari-Genomics-Lab/iDeLUCS
- name: DeLUCS
  role: Specialized DNA-clustering baseline whose ACC is compared against iDeLUCS
- name: k-means++
  role: Classical baseline run on the same k-mer frequency signature vectors for accuracy
    comparison
- name: PyTorch
  role: Machine learning framework underlying iDeLUCS training that produces the evaluated
    clusters
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad508
    title: '<i>i</i>DeLUCS: a deep learning interactive tool for alignment-free clustering
      of DNA sequences'
  source_content_hash: sha256:504e8d745591eac009d895963506fc270c024832165a01b856585c57e13a2684
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Kari-Genomics-Lab/iDeLUCS
  tools:
  - DeLUCS
  - PyTorch
  - Python
  - iDeLUCS
  - k-means++
  - pip
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# unsupervised-clustering-accuracy-evaluation

## Summary

Evaluates DNA-sequence clustering results (e.g., from iDeLUCS, DeLUCS, or k-means++) by computing unsupervised clustering accuracy (ACC) against ground-truth cluster_id labels via posthoc optimal label alignment, never using labels during training.

## When to use

Use when you have predicted cluster assignments for DNA sequences and a ground-truth TSV with sequence_id and cluster_id columns, and need to quantify clustering quality without training on labels.

## When NOT to use

- No ground-truth labels exist for the dataset — the GT_file requires cluster_id assignments; without them ACC cannot be computed.
- You need labels during training — iDeLUCS is unsupervised; the GT labels 'won't be used during training, just for posthoc analysis', so this skill must never influence clustering itself.
- Input is not DNA sequence clustering output (e.g., already a feature table or metadata matrix) — the skill evaluates cluster identifier assignments against biological truth labels.

## Inputs

- Predicted cluster assignments per sequence from a clustering run (e.g., iDeLUCS output)
- Ground-truth TSV file with sequence_id and cluster_id columns (GT_file, e.g., Example/Vertebrata_GT.tsv)
- FASTA file of DNA sequences whose identifiers correspond to the sequence_id values (e.g., Example/Vertebrata.fas)

## Outputs

- accuracy_comparison.csv with one accuracy row per method and pairwise gaps (e.g., iDeLUCS-minus-baseline)
- Posthoc unsupervised clustering ACC values per aggregation or method (e.g., acc_comparison.tsv for ensemble vs. majority voting)

## How to apply

Run the clustering method (e.g., iDeLUCS with --k=6, --n_epochs in the recommended 50-150 range, --lambda between 1 and 3, --n_clusters set to at least the number of true clusters) without passing ground-truth labels into training. Then load the GT TSV containing sequence_id and cluster_id columns and match predicted cluster identifiers to ground-truth labels via optimal label alignment (posthoc only). Tabulate one ACC row per method (e.g., iDeLUCS vs. DeLUCS vs. k-means++ with k=8 on the same k-mer signature vectors) and compute pairwise accuracy gaps to verify expected advantages, such as iDeLUCS exceeding DeLUCS by ~16% and k-means++ by ~7% on the simulated 432,333-read metagenomic dataset, or outperforming specialized algorithms by ~12% on real DNA datasets.

## Related tools

- **iDeLUCS** (Produces the cluster assignments being evaluated; PyTorch-backed deep learning clustering with contrastive loss and information-theoretic ensemble) — https://github.com/Kari-Genomics-Lab/iDeLUCS
- **DeLUCS** (Specialized DNA-clustering baseline whose ACC is compared against iDeLUCS)
- **k-means++** (Classical baseline run on the same k-mer frequency signature vectors for accuracy comparison)
- **PyTorch** (Machine learning framework underlying iDeLUCS training that produces the evaluated clusters)

## Examples

```
idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35 --lambda=2.8 --k=6 --n_clusters=5 --n_mimics=3 --batch_sz=512
```

## Evaluation signals

- GT file schema check: it must contain a sequence_id column matching FASTA identifiers and a cluster_id column of ground-truth assignments
- accuracy_comparison.csv exists with one accuracy row per method (iDeLUCS, DeLUCS, k-means++)
- Measured iDeLUCS-minus-baseline gaps fall at approximately 16% vs DeLUCS and approximately 7% vs k-means++ on the simulated metagenomic reads dataset
- The information-theoretic ensemble ACC exceeds the majority-voting baseline ACC on Vertebrata ground-truth labels
- Labels were used only posthoc — verify the clustering command produced predictions before any label alignment step

## Limitations

- GT labels are reserved for posthoc analysis only and must never enter training; misuse would invalidate the unsupervised ACC metric
- n_clusters must be set to at least n_true_clusters when a GT_file is provided; setting it below the true count will cap achievable ACC
- PyTorch's latest stable release may not be compatible with the user's CUDA version, affecting reproducibility of the clustering runs being evaluated

## Evidence

- [readme] GT labels posthoc only: "Path for a tab-separated file with possible labels for the training dataset. These labels won't be used during training, just for posthoc analysis."
- [readme] GT file required columns: [evidence span withheld — anchor status: not_found]
- [readme] n_clusters constraint: "It should be at least `n_true_clusters` when `GT_file` is provided"
- [abstract] Accuracy advantage finding: "iDeLUCS outperforms the two classical algorithms by an average of ∼20%, and the two specialized algorithms by an average of ∼12%, on the datasets of real DNA sequences analyzed"
- [methods] Ground-truth evaluation example: "idelucs --sequence_file=Example/Vertebrata.fas --GT_file=Example/Vertebrata_GT.tsv --n_epochs=35"
