---
name: correlation-distance-hierarchical-clustering
description: Use when you have labeled or unlabeled protein FASTA families and want
  to test whether AAR-kmer similarity recovers known family structure (e.g., nitrogen
  cycling families) without training supervised models.
when_to_use_negative:
- You already have trusted family assignments and only want to annotate new sequences
  — use Snekmer's 'apply'/'search' modes with pre-trained models instead of de novo
  clustering
- You need probabilistic per-sequence annotations with scores (Prediction/Score/delta/Confidence)
  — that is the supervised model/apply pipeline, not cluster mode
- The input is not protein sequences (e.g., nucleotide or already-vectorized feature
  tables) — Snekmer expects FASTA protein sequences for AAR-kmer conversion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Snekmer
  role: 'Runs the ''cluster'' operation mode: converts protein sequences into AAR-kmer
    vectors and performs hierarchical clustering with correlation distance; sweeps
    alphabets 0-5 plus None/size-20 and k-mer lengths from config.yaml'
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Workflow engine underlying Snekmer; orchestrates the cluster-mode pipeline
    steps when 'snekmer cluster' is invoked
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runtime (3.11+) for Snekmer and for computing per-family ROC AUC comparisons
    of cluster memberships against HMM ground-truth labels
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:bd91c0a7322ffc6c09c2df0a715bc4c0907f5407ed10faeaa281203b7951a8c8
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
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# correlation-distance-hierarchical-clustering

## Summary

Snekmer's unsupervised cluster mode groups protein sequences by hierarchically clustering AAR kmer vectors with correlation distance, enabling de novo family determination that can be validated against HMM-derived ground-truth labels via per-family ROC AUC.

## When to use

Use when you have labeled or unlabeled protein FASTA families and want to test whether AAR-kmer similarity recovers known family structure (e.g., nitrogen cycling families) without training supervised models.

## When NOT to use

- You already have trusted family assignments and only want to annotate new sequences — use Snekmer's 'apply'/'search' modes with pre-trained models instead of de novo clustering
- You need probabilistic per-sequence annotations with scores (Prediction/Score/delta/Confidence) — that is the supervised model/apply pipeline, not cluster mode
- The input is not protein sequences (e.g., nucleotide or already-vectorized feature tables) — Snekmer expects FASTA protein sequences for AAR-kmer conversion

## Inputs

- Protein family FASTA files (e.g., nitrogen cycling families)
- Ground-truth family labels from original HMM family designations
- config.yaml specifying alphabets, k-mer lengths, linkage, and distance_threshold in cluster.params

## Outputs

- Cluster memberships of AAR-kmer vectors per alphabet/kmer-length combination
- Per-family ROC AUC values for each alphabet/kmer-length combination
- Averaged per-combination AUC summary heatmap (cluster_plots enabled)

## How to apply

Place the protein family FASTA files (e.g., nitrogen cycling families with ground-truth HMM labels) into the Snekmer input directory, then run 'snekmer cluster' to convert sequences into AAR-kmer vectors while sweeping recoding alphabets 0-5 (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs, plus the None/size-20 alphabet) and k-mer lengths set in config.yaml. Cluster the resulting kmer vectors using the 'correlation' method, i.e., hierarchical clustering with correlation distance, governed by the linkage and distance_threshold settings under cluster.params. Compare cluster memberships to the HMM ground-truth labels and compute per-family ROC AUC for every alphabet/kmer-length combination, then average per-family AUCs per combination. Enable cluster_plots to render the summary AUC heatmap; in the published validation, average ROC AUC exceeded 0.9 for more than half of the alphabet/kmer-length combinations, matching the original HMM performance (Figure 4).

## Related tools

- **Snekmer** (Runs the 'cluster' operation mode: converts protein sequences into AAR-kmer vectors and performs hierarchical clustering with correlation distance; sweeps alphabets 0-5 plus None/size-20 and k-mer lengths from config.yaml) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Workflow engine underlying Snekmer; orchestrates the cluster-mode pipeline steps when 'snekmer cluster' is invoked) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Runtime (3.11+) for Snekmer and for computing per-family ROC AUC comparisons of cluster memberships against HMM ground-truth labels) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
snekmer cluster
```

## Evaluation signals

- Reproduced summary heatmap shows average ROC AUC > 0.9 in more than half of the alphabet/kmer-length combinations, matching reported Figure 4 clustering performance
- Cluster assignments are produced for every swept alphabet (0-5 plus None/size-20) and each configured k-mer length without missing combinations
- Per-family ROC AUC values are computed against the HMM ground-truth labels for each family, not just global clustering metrics
- Snekmer completes without errors: 'snekmer -h' verifies installation and the Snakemake-driven cluster workflow runs to completion on the input FASTA directory

## Limitations

- Exact-match kmer representations 'may be limited in terms of sensitivity and coverage'; AAR recoding is used to link sequences with distant sequence similarity, but performance varies across alphabet/kmer-length combinations
- Cluster-mode agreement with HMM labels is family-dependent — AUC > 0.9 held for more than half, not all, alphabet/kmer-length combinations
- Clustering depends on user-specified linkage and distance_threshold in cluster.params; poor choices may fragment or merge families

## Evidence

- [results] Snekmer's unsupervised mode reproduces the HMM-based nitrogen cycling family structure well: average ROC AUC exceeds 0.9: [evidence span withheld — anchor status: not_found]
- [methods] Cluster the kmer vectors with the 'correlation' method (hierarchical clustering using correlation distance) using the linkage and distance_threshold: [evidence span withheld — anchor status: not_found]
- [methods] sweeping the recoding alphabets 0-5 (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs, plus None/size-20) and k-mer lengths specified via config.yaml: [evidence span withheld — anchor status: not_found]
- [intro] Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms, (2) generates supervised machine learning models, or (3) searches sequences against pre-trained models to"
- [readme] There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`, plus a guided `easy` mode that runs the Learn/Apply pipeline from a single command."
