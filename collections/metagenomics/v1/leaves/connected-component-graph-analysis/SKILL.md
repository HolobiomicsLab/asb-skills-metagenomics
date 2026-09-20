---
name: connected-component-graph-analysis
description: Use when a pairwise protein-protein correlation/distance matrix (e.g.,
  Snekmer cluster mode with save_matrix) must be binarized at a threshold like 0.5
  and decomposed into connected components to verify family-level subnetwork separation.
when_to_use_negative:
- The goal is supervised family prediction with scores — use Snekmer's model/search/apply
  modes, which output snekmer_results.csv with Prediction, Score, delta, and Confidence
  columns instead of a thresholded network
- Input is raw protein sequences (e.g., FASTA) rather than a saved pairwise correlation/distance
  matrix — the AAR-kmer vectorization (Snekmer cluster mode) must run first
- A partition-based clustering result (agglomerative or DBSCAN with Jaccard distance)
  is needed directly rather than threshold-based graph subnetworks
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Generates the pairwise protein-protein correlation/distance matrix via unsupervised
    cluster mode (save_matrix) and supports graph-figure rendering of clustering results
    (cluster_plots)
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: bsf-jaccard-py
  role: Blazing Signature Filter Jaccard similarity backend used to compute pairwise
    similarities efficiently; falls back to scipy when BSF is unavailable
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Python
  role: Language in which the thresholding, connected-component computation, and graph
    rendering are implemented
- name: Snakemake
  role: Orchestrates the Snekmer cluster workflow steps that produce the correlation
    matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:78aa4cf6d810321d417a10f98688c7998e64b3535015aca043659ee202c387fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# connected-component-graph-analysis

## Summary

Thresholds the Snekmer protein-protein correlation/distance matrix (from unsupervised cluster mode) into a network graph and computes connected components to test whether pre-annotated families separate into distinct, unconnected subnetworks at conservative correlation cutoffs.

## When to use

Use when a pairwise protein-protein correlation/distance matrix (e.g., Snekmer cluster mode with save_matrix) must be binarized at a threshold like 0.5 and decomposed into connected components to verify family-level subnetwork separation.

## When NOT to use

- The goal is supervised family prediction with scores — use Snekmer's model/search/apply modes, which output snekmer_results.csv with Prediction, Score, delta, and Confidence columns instead of a thresholded network
- Input is raw protein sequences (e.g., FASTA) rather than a saved pairwise correlation/distance matrix — the AAR-kmer vectorization (Snekmer cluster mode) must run first
- A partition-based clustering result (agglomerative or DBSCAN with Jaccard distance) is needed directly rather than threshold-based graph subnetworks

## Inputs

- Pairwise protein-protein correlation/distance matrix produced by Snekmer cluster mode (save_matrix: True)
- Correlation threshold value (e.g., 0.5 for nitrogen-family networks; 0.1 for solvent accessibility alphabet, k=12)
- Pre-annotated protein family assignments used to validate subnetwork separation
- Encoding configuration (e.g., alphabet 2 = solvacc, k-mer length k=12)

## Outputs

- Thresholded network graph (binarized adjacency of protein pairs above the correlation cutoff)
- Connected-component assignment per protein (subnetwork membership labels)
- Graph figures of the thresholded networks (per Snekmer cluster_plots behavior)

## How to apply

Run Snekmer in unsupervised cluster mode with save_matrix enabled so the pairwise protein-protein correlation/distance matrix is written to disk. Binarize the matrix into network edges by keeping protein pairs with correlation above the chosen threshold — 0.5 for the nitrogen-cycle family analysis, 0.1 for the solvent accessibility alphabet (alphabet 2 = solvacc, k=12) encoding. Compute connected components on the thresholded graph and assign each protein to its component. Render the thresholded networks as graph figures, consistent with Snekmer's cluster_plots option for illustrating clustering results. Validate by confirming that at correlation threshold 0.5 the pre-annotated protein families map onto distinct, unconnected subnetworks (no edges between families), matching the figure-5 behavior described for nitrogen-family clustering; the solvacc k=12 network at threshold 0.1 should show similar clustering at reduced computational cost.

## Related tools

- **Snekmer** (Generates the pairwise protein-protein correlation/distance matrix via unsupervised cluster mode (save_matrix) and supports graph-figure rendering of clustering results (cluster_plots)) — https://github.com/PNNL-CompBio/Snekmer
- **bsf-jaccard-py** (Blazing Signature Filter Jaccard similarity backend used to compute pairwise similarities efficiently; falls back to scipy when BSF is unavailable) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Python** (Language in which the thresholding, connected-component computation, and graph rendering are implemented)
- **Snakemake** (Orchestrates the Snekmer cluster workflow steps that produce the correlation matrix)

## Examples

```
from scipy.sparse.csgraph import connected_components; n, labels = connected_components((corr > 0.5).astype(int), directed=False)
```

## Evaluation signals

- At correlation threshold 0.5, no edges exist between pre-annotated nitrogen-cycle protein families — connected components map 1:1 onto family labels
- The solvent accessibility alphabet (k=12) network at threshold 0.1 yields clustering patterns similar to the nitrogen-family threshold-0.5 result
- Every protein in the input matrix is assigned to exactly one connected component (labels form a complete partition)
- The binarized adjacency is symmetric with a zero diagonal (self-edges removed) before component computation
- Rendered graph figures show distinct, unconnected subnetworks consistent with the figure-5 nitrogen-family clustering behavior

## Limitations

- Snekmer's kmer set representation uses an exact-match methodology that may be limited in sensitivity and coverage; amino acid recoding links sequences with distant sequence similarity but does not remove this constraint
- Threshold choice is decisive: the 0.5 cutoff is deliberately conservative; lower thresholds risk merging families while higher ones may fragment them — the analogous solvacc k=12 network requires a much lower threshold of 0.1
- The Jaccard-distance clustering backend requires BSF, which needs a C/C++ toolchain (GCC 4.9+ for OpenMP/C++11); without it the pipeline falls back to scipy
- Requires Python 3.11+ per the Snekmer README

## Evidence

- [results] Varying the correlation threshold shows that at a more conservative correlation value of 0.5 the nitrogen-cycle families form distinct, unconnected subnetworks: [evidence span withheld — anchor status: not_found]
- [methods] Load the pairwise protein-protein correlation/distance matrix produced by Snekmer cluster mode with the save_matrix option enabled: [evidence span withheld — anchor status: not_found]
- [readme] Snekmer clusters sequences using unsupervised algorithms from AAR-kmer representations: "Snekmer is a software package designed to reduce the representation of protein sequences
by combining amino acid reduction (AAR) with the kmer approach. Based on the AAR-kmer representations,
Snekmer"
- [methods] Jaccard agglomerative clustering requires BSF or falls back to scipy: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [abstract] Snekmer accurately differentiates nitrogen cycling families in supervised and unsupervised modes: [evidence span withheld — anchor status: not_found]
