---
name: per-family-roc-auc-evaluation
description: Use when you need to quantify agreement between de novo clustering results
  (e.g. Snekmer cluster output) and known ground-truth family labels, per family,
  across a parameter sweep.
when_to_use_negative:
- Input sequences lack any ground-truth family labels — ROC AUC against HMM designations
  cannot be computed
- The goal is supervised classification or annotation scoring (use Snekmer model/search/apply
  modes instead of cluster + AUC evaluation)
- Input is already a feature/cluster-assignment table rather than raw protein FASTA
  sequences
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Runs the cluster mode to generate AAR-kmer vectors and hierarchical (correlation)
    clustering across alphabet/k-mer sweeps
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Workflow engine orchestrating the Snekmer cluster pipeline steps
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Used (Python 3.11+) to compute per-family ROC AUC against HMM ground-truth
    labels and render the summary heatmap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:635a7d55290040b3d820bb24a8c9982cbd88eb4783023683d7c2dd8d2a627abf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# per-family-roc-auc-evaluation

## Summary

Evaluates how well Snekmer's unsupervised AAR-kmer clustering of nitrogen cycling protein families reproduces the original HMM-derived family designations by computing per-family ROC AUC for each alphabet/kmer-length combination and averaging them into a summary heatmap.

## When to use

Use when you need to quantify agreement between de novo clustering results (e.g. Snekmer cluster output) and known ground-truth family labels, per family, across a parameter sweep.

## When NOT to use

- Input sequences lack any ground-truth family labels — ROC AUC against HMM designations cannot be computed
- The goal is supervised classification or annotation scoring (use Snekmer model/search/apply modes instead of cluster + AUC evaluation)
- Input is already a feature/cluster-assignment table rather than raw protein FASTA sequences

## Inputs

- Nitrogen cycling family FASTA files (protein sequences)
- Ground-truth family labels from original HMM family designations
- config.yaml specifying k-mer lengths, alphabets, and cluster.params (linkage, distance_threshold)

## Outputs

- Per-family ROC AUC values for each alphabet/kmer-length combination
- Averaged per-combination AUC summary
- Summary heatmap of clustering performance (via cluster_plots)

## How to apply

Attach ground-truth labels taken from the original HMM family designations to the nitrogen cycling family FASTA files placed in the Snekmer input directory. Run snekmer cluster to convert sequences into AAR-kmer vectors while sweeping recoding alphabets 0-5 (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs) plus None/size-20, and the k-mer lengths specified in config.yaml. Cluster the kmer vectors with the 'correlation' method (hierarchical clustering using correlation distance), using the linkage and distance_threshold settings in cluster.params. Compare resulting cluster memberships to the HMM ground-truth labels and compute ROC AUC per family for each alphabet/kmer-length combination, then average the per-family AUCs per parameter combination. Render the summary heatmap (cluster_plots enabled) and validate that average AUC exceeds 0.9 in more than half of the alphabet/kmer-length combinations, matching the reported Figure 4 clustering performance.

## Related tools

- **Snekmer** (Runs the cluster mode to generate AAR-kmer vectors and hierarchical (correlation) clustering across alphabet/k-mer sweeps) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Workflow engine orchestrating the Snekmer cluster pipeline steps) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Used (Python 3.11+) to compute per-family ROC AUC against HMM ground-truth labels and render the summary heatmap)

## Examples

```
snekmer cluster
```

## Evaluation signals

- Reproduced heatmap shows average ROC AUC greater than 0.9 in more than half of the alphabet/kmer-length combinations, matching the reported Figure 4 clustering performance
- Per-family ROC AUC values are computed for every family and every alphabet (0-5 and None/size-20) / k-mer length combination with no missing cells
- Cluster memberships are derived with the 'correlation' method (hierarchical clustering using correlation distance) and the linkage/distance_threshold settings from cluster.params
- Ground-truth labels used for AUC calculation match the original HMM family designations for all input sequences

## Limitations

- Representing proteins as kmer sets uses an exact-match methodology that may be limited in sensitivity and coverage; AAR is used to link sequences with distant sequence similarity
- Average AUC > 0.9 is achieved for more than half — not all — alphabet/kmer-length combinations, so parameter choice materially affects agreement with HMM labels

## Evidence

- [abstract] Snekmer's unsupervised mode reproduces the HMM-based nitrogen cycling family structure well: [evidence span withheld — anchor status: not_found]
- [intro] cluster mode purpose: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
- [readme] operation modes: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
- [abstract] kmer exact-match limitation: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
