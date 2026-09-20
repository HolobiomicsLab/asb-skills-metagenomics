---
name: correlation-distance-network-construction
description: Use when Snekmer cluster mode (unsupervised AAR-kmer clustering) has
  produced a saved pairwise protein-protein correlation/distance matrix and you need
  to verify, at a chosen correlation threshold, that known protein families form unconnected
  subnetworks.
when_to_use_negative:
- Input is raw protein FASTA that has not yet been processed by Snekmer cluster mode
  — there is no pairwise correlation/distance matrix to threshold
- The analysis is supervised classification (Snekmer model/search/learn/apply outputs
  such as results/apply/snekmer_results.csv with cosine-similarity Prediction/Score/delta
  columns) rather than unsupervised clustering — the apply-step CSV is not a network
  matrix
- Only a single family or already-grouped sequence set is available, so cross-family
  separation cannot be assessed
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Snekmer
  role: Generates the pairwise protein-protein correlation/distance matrix in cluster
    mode via the ``save_matrix`` option and renders clustering figures via ``cluster_plots``
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: bsf-jaccard-py
  role: Provides the Blazing Signature Filter Jaccard pairwise similarity computation
    underlying the cluster-mode distance matrix
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Python
  role: Used to binarize the thresholded matrix into edges and compute connected components
    on the resulting graph
- name: Snakemake
  role: Workflow engine through which Snekmer cluster mode (and its saved matrix output)
    is executed
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:ff8b91abe0f9ae67ba55d8d1178da1798d50f2ca93788855416c7ae587f0db0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# correlation-distance-network-construction

## Summary

Thresholds the pairwise protein-protein correlation/distance matrix saved by Snekmer cluster mode (``save_matrix``) into an edge list, then computes connected components to test whether pre-annotated protein families separate into distinct, unconnected subnetworks at a conservative correlation value (0.5 for nitrogen-cycle families, 0.1 for the solvent accessibility alphabet with k=12).

## When to use

Use when Snekmer cluster mode (unsupervised AAR-kmer clustering) has produced a saved pairwise protein-protein correlation/distance matrix and you need to verify, at a chosen correlation threshold, that known protein families form unconnected subnetworks.

## When NOT to use

- Input is raw protein FASTA that has not yet been processed by Snekmer cluster mode — there is no pairwise correlation/distance matrix to threshold
- The analysis is supervised classification (Snekmer model/search/learn/apply outputs such as results/apply/snekmer_results.csv with cosine-similarity Prediction/Score/delta columns) rather than unsupervised clustering — the apply-step CSV is not a network matrix
- Only a single family or already-grouped sequence set is available, so cross-family separation cannot be assessed

## Inputs

- Pairwise protein-protein correlation/distance matrix from Snekmer cluster mode run with ``save_matrix``: True
- Protein family annotations for the input sequences (e.g. TIGRFAMs-style .ann annotations) used to label components
- Amino acid recoded (AAR) k-mer representations of the input proteins (e.g. solvacc alphabet 2, k=12 for the solvent-accessibility variant)

## Outputs

- Binarized edge list / thresholded network graph connecting protein pairs above the correlation threshold
- Connected-component assignments mapping each protein to its subnetwork
- Graph figures of the thresholded networks (in the style of Snekmer ``cluster_plots`` output)

## How to apply

Load the pairwise protein-protein correlation/distance matrix produced by Snekmer cluster mode with ``save_matrix`` enabled; this matrix derives from the unsupervised clustering side of Snekmer (agglomerative clustering using Jaccard distance by default, accelerated by the Blazing Signature Filter with a scipy fallback). Binarize the matrix into network edges so that protein pairs with correlation above the threshold are connected — use a conservative correlation value of 0.5 for the nitrogen-cycle families. Compute connected components on the thresholded graph and assign each protein to its component. Repeat the thresholding for the solvent accessibility encoding (alphabet 2 = solvacc, k-mer length k=12) at threshold 0.1 to check whether similar clustering is reproduced at reduced computational cost. Render the thresholded networks as graph figures, consistent with Snekmer's ``cluster_plots`` option that generates figures illustrating clustering results. The decision point is threshold selection: too low a value merges families into one giant component, so 0.5 is used for the nitrogen-family demonstration.

## Related tools

- **Snekmer** (Generates the pairwise protein-protein correlation/distance matrix in cluster mode via the ``save_matrix`` option and renders clustering figures via ``cluster_plots``) — https://github.com/PNNL-CompBio/Snekmer
- **bsf-jaccard-py** (Provides the Blazing Signature Filter Jaccard pairwise similarity computation underlying the cluster-mode distance matrix) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Python** (Used to binarize the thresholded matrix into edges and compute connected components on the resulting graph)
- **Snakemake** (Workflow engine through which Snekmer cluster mode (and its saved matrix output) is executed) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
import networkx as nx; G = nx.Graph(); G.add_edges_from((a, b) for a, b, c in protein_pairs if c >= 0.5); components = list(nx.connected_components(G))
```

## Evaluation signals

- At correlation threshold 0.5, connected-component analysis yields subnetworks with zero edges between pre-annotated nitrogen-cycle protein families (each component is family-pure)
- The solvent accessibility alphabet (alphabet 2 = solvacc, k=12) network at threshold 0.1 reproduces the same family separation behavior
- Each protein in the input appears exactly once across the connected components, and the number of components at threshold 0.5 is at least the number of distinct annotated families
- The thresholded graph is derived from a square, symmetric pairwise matrix with one entry per input protein
- Rendered network figures visually match the described figure-5 behavior of distinct, unconnected family subnetworks

## Limitations

- Representing proteins as kmer sets uses an exact-match methodology that may be limited in terms of sensitivity and coverage, so the correlation matrix may miss distant relationships
- Threshold choice is critical and family-dependent: 0.5 is described as conservative for the nitrogen-cycle families; a lower value risks merging families into one connected component
- The default agglomerative Jaccard clustering requires BSF (a C/C++ extension needing OpenMP and GCC 4.9+); without it Snekmer falls back to scipy
- Similar clustering at reduced computational cost (solvacc, k=12) trades encoding richness for speed

## Evidence

- [readme] Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach.: "Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach."
- [methods] Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy): "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [abstract] a set of nitrogen cycling families originally collected using both standard hidden Markov models and a larger set of proteins from Uniprot: "a set of nitrogen cycling families originally collected using both standard hidden Markov models and a larger set of proteins from Uniprot"
- [readme] The Blazing Signature Filter (BSF) is a highly efficient pairwise similarity algorithm which enables extensive data mining within a reasonable amount of time.: [evidence span withheld — anchor status: external_doc]
- [readme] Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
