---
name: parameter-sweep-heatmap-visualization
description: Use when benchmarking Snekmer's unsupervised cluster mode across recoding
  alphabet (0-5 plus None/size-20) and kmer length combinations against known HMM-derived
  family labels, and rendering the AUC results as a summary heatmap.
when_to_use_negative:
- Input sequences lack trustworthy family labels, so per-family ROC AUC against HMM
  ground truth cannot be computed (de novo clustering alone is not scored by this
  skill)
- The goal is supervised classification or searching against pre-trained Snekmer models
  rather than unsupervised cluster-mode evaluation
- Input is already a feature/kmer-vector table from a prior Snekmer run with no planned
  re-sweep of alphabets or kmer lengths
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snekmer
  role: Recodes proteins into AAR-kmer vectors, runs unsupervised clustering with
    the 'correlation' method across alphabet and kmer length settings, and renders
    the summary heatmap via cluster_plots
  repo: github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the cluster-mode workflow steps executed by Snekmer
  repo: github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runtime environment (3.11+) in which Snekmer and its ROC AUC/plotting dependencies
    are installed and run
  repo: github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:cb13e2df5fa5a5599e81dc1b0342539f56e36ba735978217724c91a7e4c9c148
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# parameter-sweep-heatmap-visualization

## Summary

Sweep Snekmer's unsupervised clustering over amino-acid recoding alphabets and kmer lengths, score each parameter combination against ground-truth labels with per-family ROC AUC, and summarize the grid as a heatmap to validate clustering fidelity.

## When to use

Use when benchmarking Snekmer's unsupervised cluster mode across recoding alphabet (0-5 plus None/size-20) and kmer length combinations against known HMM-derived family labels, and rendering the AUC results as a summary heatmap.

## When NOT to use

- Input sequences lack trustworthy family labels, so per-family ROC AUC against HMM ground truth cannot be computed (de novo clustering alone is not scored by this skill)
- The goal is supervised classification or searching against pre-trained Snekmer models rather than unsupervised cluster-mode evaluation
- Input is already a feature/kmer-vector table from a prior Snekmer run with no planned re-sweep of alphabets or kmer lengths

## Inputs

- Nitrogen cycling family FASTA files
- Ground-truth per-sequence labels from original HMM family designations
- config.yaml specifying alphabets (0-5, None/size-20), kmer lengths, and cluster.params (method=correlation, linkage, distance_threshold)

## Outputs

- AAR-kmer vector representations per sequence per alphabet/kmer-length combination
- Cluster memberships for each parameter combination
- Per-family ROC AUC values averaged over each alphabet/kmer-length grid cell
- Summary heatmap of average AUC across the alphabet-by-kmer-length grid (cluster_plots output)

## How to apply

Place the nitrogen cycling family FASTA files into the Snekmer input directory and attach ground-truth labels from the original HMM family designations. Run 'snekmer cluster' to convert sequences into AAR-kmer vectors, sweeping alphabets 0-5 (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs) plus None/size-20 and kmer lengths defined in config.yaml (search params are set in the provided example config). Cluster each kmer-vector set hierarchically with the 'correlation' method, applying the linkage and distance_threshold settings under cluster.params. For each alphabet/kmer-length combination, compare cluster memberships to the HMM ground-truth labels and compute per-family ROC AUC, then average the per-family AUCs; enable cluster_plots to render the summary heatmap. The skill is validated when the reproduced heatmap shows average AUC greater than 0.9 in more than half of the alphabet/kmer-length combinations, matching the paper's Figure 4 clustering performance.

## Related tools

- **Snekmer** (Recodes proteins into AAR-kmer vectors, runs unsupervised clustering with the 'correlation' method across alphabet and kmer length settings, and renders the summary heatmap via cluster_plots) — github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the cluster-mode workflow steps executed by Snekmer) — github.com/PNNL-CompBio/Snekmer
- **Python** (Runtime environment (3.11+) in which Snekmer and its ROC AUC/plotting dependencies are installed and run) — github.com/PNNL-CompBio/Snekmer

## Examples

```
snekmer cluster
```

## Evaluation signals

- The reproduced heatmap shows average ROC AUC greater than 0.9 in more than half of the examined alphabet and kmer length combinations
- Per-family AUCs are computed for every (alphabet, kmer-length) grid cell against the HMM ground-truth labels
- Cluster mode used the 'correlation' method (hierarchical clustering with correlation distance) with the configured linkage and distance_threshold
- All alphabets (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs, and None/size-20) and configured kmer lengths appear in the sweep grid
- Generated heatmap matches the performance pattern reported in the article's Figure 4

## Limitations

- Exact-match kmer representation may be limited in sensitivity and coverage; AAR recoding links sequences with distant sequence similarity, so performance varies across alphabet choices
- Validation threshold (>0.9 average AUC in >half of combinations) was established specifically for the nitrogen cycling families and may not transfer to other family sets
- BSF-based clustering requires installing bsf-jaccard-py (with a C/C++ toolchain, GCC 4.9+ for OpenMP) or falls back to scipy

## Evidence

- [abstract] Unsupervised mode reproduces HMM family structure: [evidence span withheld — anchor status: not_found]
- [readme] Unsupervised clustering of AAR-kmer representations: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms, (2) generates supervised machine learning models, or (3) searches sequences against pre-trained models"
- [intro] Cluster mode is one of six operation modes: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
- [results] Config example available for parameter settings: "An example ``config.yaml`` is included at `resources/config.yaml <https://github.com/PNNL-CompBio/Snekmer/blob/main/resources/config.yaml>`_"
