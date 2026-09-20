---
name: kmer-vector-distance-computation
description: Use when protein sequences have been recoded into AAR-kmer vectors and
  you need a pairwise correlation/distance matrix, thresholded into a graph, to test
  whether protein families form unconnected subnetworks.
when_to_use_negative:
- When the goal is supervised family classification rather than de novo unsupervised
  separation — use Snekmer's model/learn/apply modes, which output snekmer_results.csv
  with Prediction/Score/delta columns instead of a distance matrix
- When sequences are better analyzed by exact kmer matching without amino acid recoding
  — the article notes exact-match kmer sets 'may be limited in terms of sensitivity
  and coverage' and AAR exists precisely to link distant-sequence-similarity proteins
- When the correlation matrix already exists and no thresholding/network analysis
  is needed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Recodes proteins into AAR-kmer vectors and, in cluster mode with save_matrix,
    produces the pairwise protein-protein correlation/distance matrix; cluster_plots
    generates the clustering figures
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: bsf-jaccard-py
  role: Blazing Signature Filter Python wrapper providing fast pairwise Jaccard similarity
    over kmer vectors used by Snekmer's agglomerative-jaccard and density-jaccard
    clustering
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Snakemake
  role: Workflow engine orchestrating the Snekmer cluster pipeline; Snakemake arguments
    are passed through directly
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:cb2c1e073bdd53ba0ca6050090b02b39cd77d953cd9860594c26d17086423e3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kmer-vector-distance-computation

## Summary

Compute pairwise protein-protein distance/similarity matrices over amino-acid-recoded (AAR) kmer vectors with Snekmer's cluster mode, then threshold the matrix into network graphs where pre-annotated families separate into distinct subnetworks.

## When to use

Use when protein sequences have been recoded into AAR-kmer vectors and you need a pairwise correlation/distance matrix, thresholded into a graph, to test whether protein families form unconnected subnetworks.

## When NOT to use

- When the goal is supervised family classification rather than de novo unsupervised separation — use Snekmer's model/learn/apply modes, which output snekmer_results.csv with Prediction/Score/delta columns instead of a distance matrix
- When sequences are better analyzed by exact kmer matching without amino acid recoding — the article notes exact-match kmer sets 'may be limited in terms of sensitivity and coverage' and AAR exists precisely to link distant-sequence-similarity proteins
- When the correlation matrix already exists and no thresholding/network analysis is needed

## Inputs

- Protein sequences in FASTA format (e.g., nitrogen cycling families from TIGRFAM/HMM collections and Uniprot proteins)
- Snekmer AAR-kmer vector representations (kmer length configurable, e.g., k=12 for the solvent accessibility alphabet)
- Snekmer config.yaml specifying alphabet, k-mer length, clustering method, and save_matrix: True
- Pairwise protein-protein correlation/distance matrix emitted by Snekmer cluster mode

## Outputs

- Thresholded network edge list (pairs with correlation > 0.5, or > 0.1 for solvacc k=12)
- Connected-component assignments mapping each protein to its subnetwork
- Network graph figures illustrating clustering results (cluster_plots)
- Saved pairwise distance/correlation matrix file

## How to apply

Run Snekmer in cluster mode with ``save_matrix`` enabled so the pairwise protein-protein correlation/distance matrix over AAR-kmer vectors is written to disk (Snekmer's Jaccard clustering uses the Blazing Signature Filter via bsf-jaccard-py, falling back to scipy). Binarize the matrix into edges at the correlation threshold used in the nitrogen-family analysis (0.5; 0.1 for the solvent accessibility alphabet with k=12), connecting protein pairs with correlation above the cutoff. Compute connected components on the thresholded graph and assign each protein to a component, then render network figures consistent with Snekmer's cluster_plots option. Validate that at threshold 0.5 the pre-annotated nitrogen-cycle families map onto distinct, unconnected subnetworks with no inter-family edges, mirroring the figure-5 behavior.

## Related tools

- **Snekmer** (Recodes proteins into AAR-kmer vectors and, in cluster mode with save_matrix, produces the pairwise protein-protein correlation/distance matrix; cluster_plots generates the clustering figures) — https://github.com/PNNL-CompBio/Snekmer
- **bsf-jaccard-py** (Blazing Signature Filter Python wrapper providing fast pairwise Jaccard similarity over kmer vectors used by Snekmer's agglomerative-jaccard and density-jaccard clustering) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Snakemake** (Workflow engine orchestrating the Snekmer cluster pipeline; Snakemake arguments are passed through directly) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
import networkx as nx, numpy as np; G = nx.Graph(); G.add_edges_from(zip(*np.where(C > 0.5))); comps = list(nx.connected_components(G))
```

## Evaluation signals

- At correlation threshold 0.5, connected components on the thresholded graph contain no edges between pre-annotated nitrogen-cycle families (families map to distinct, unconnected subnetworks)
- The saved pairwise distance matrix (save_matrix=True) is symmetric and covers all input protein pairs
- Repeat with the solvent accessibility alphabet (alphabet 2, k=12) at threshold 0.1 yields similar family clustering, confirming robustness of the subnetwork separation
- Component membership recapitulates the figure-5 nitrogen-family clustering behavior described in the article
- Network figures render cleanly, consistent with Snekmer's cluster_plots option

## Limitations

- Exact-match kmer representation 'may be limited in terms of sensitivity and coverage'; AAR recoding mitigates but does not eliminate this
- Snekmer's Jaccard clustering with BSF requires the bsf-jaccard-py C extension (GCC 4.9+ with OpenMP) or falls back to scipy
- Threshold choice is conservative by design: at 0.5 families separate, but lower thresholds may merge subnetworks; the alternative k=12/solvacc setting trades encoding fidelity for reduced computational cost

## Evidence

- [intro] cluster_sequences_unsupervised: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
- [methods] jaccard_clustering: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [readme] bsf_pairwise: [evidence span withheld — anchor status: external_doc]
- [abstract] aar_rationale: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
