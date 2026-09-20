---
name: protein-sequence-clustering
description: Use when given unannotated multi-FASTA protein input and the goal is
  de novo family determination via unsupervised clustering of AAR-kmer representations,
  e.g. reproducing nitrogen-cycle family separation at a 0.5 correlation threshold.
when_to_use_negative:
- You already have trusted family labels and need supervised prediction — use Snekmer's
  model/search or learn/apply (easy) modes instead, which output results/apply/snekmer_results.csv
  with Prediction, Score, delta, and Confidence columns
- You need pairwise similarity of full-length sequences rather than AAR-kmer fingerprints
  — the exact-match kmer methodology may be limited in sensitivity and coverage compared
  with alignment-based search
- Input is a feature/metadata table rather than protein sequences — Snekmer requires
  FASTA sequence input
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Recodes proteins into AAR kmer vectors and runs unsupervised `cluster` mode,
    saving the pairwise distance matrix (save_matrix) and generating clustering figures
    (cluster_plots)
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Workflow engine underlying Snekmer; Snakemake arguments are passed through
    directly to Snakemake
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: bsf-jaccard-py
  role: Blazing Signature Filter Python wrapper computing pairwise Jaccard similarity,
    used as the fast backend for agglomerative/density Jaccard clustering
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Python
  role: Binarizes the thresholded correlation matrix, computes connected components,
    and assembles per-threshold summaries (e.g. with pandas)
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:f61b2a2f83841bb7decd7ec4a54ffd3fe3a843bdbcd189a8ecfbbd490d21ba7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - MMseqs2
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# protein-sequence-clustering

## Summary

De novo clustering of protein families by recoding sequences into amino acid-reduced (AAR) kmer vectors with Snekmer's cluster mode, then thresholding the saved pairwise correlation/distance matrix into network graphs whose connected components correspond to protein families.

## When to use

Use when given unannotated multi-FASTA protein input and the goal is de novo family determination via unsupervised clustering of AAR-kmer representations, e.g. reproducing nitrogen-cycle family separation at a 0.5 correlation threshold.

## When NOT to use

- You already have trusted family labels and need supervised prediction — use Snekmer's model/search or learn/apply (easy) modes instead, which output results/apply/snekmer_results.csv with Prediction, Score, delta, and Confidence columns
- You need pairwise similarity of full-length sequences rather than AAR-kmer fingerprints — the exact-match kmer methodology may be limited in sensitivity and coverage compared with alignment-based search
- Input is a feature/metadata table rather than protein sequences — Snekmer requires FASTA sequence input

## Inputs

- Multi-species protein FASTA file of sequences (e.g. nitrogen-cycle families or Uniprot proteins)
- config.yaml specifying alphabet, k-mer length (e.g. solvacc with k=12), save_matrix, and clustering options

## Outputs

- Pairwise protein-protein correlation/distance matrix (saved via the save_matrix option)
- Connected-component (subnetwork) assignments per protein at a chosen correlation threshold (0.5 nitrogen families; 0.1 solvacc k=12)
- Rendered thresholded network graph figures via the cluster_plots option

## How to apply

Run Snekmer's `cluster` mode (one of six operation modes alongside `model`, `search`, `learn`, `apply`, and `motif`) on the input FASTA with a `config.yaml` that selects the amino acid recoding alphabet and k-mer length — e.g. the solvent accessibility alphabet (alphabet 2 = solvacc) with k=12, which achieved similar clustering to the default at reduced computational cost. Enable `save_matrix` so the pairwise protein-protein correlation/distance matrix is written to disk, and optionally set `min_rep` to discard k-mers with fewer than that many occurrences across the input set. By default Snekmer uses agglomerative clustering with Jaccard distance (the BSF implementation via bsf-jaccard-py speeds this up; otherwise it falls back to scipy). To reproduce the network analysis, binarize the saved correlation matrix at the conservative threshold of 0.5 (0.1 for the solvacc k=12 network) so pairs above the threshold become edges, compute connected components, and render graph figures in line with the `cluster_plots` option. Correct application is confirmed when pre-annotated families (e.g. nitrogen-cycle families) map onto distinct, unconnected subnetworks with no inter-family edges, matching the figure-5 behavior reported for nitrogen-family clustering.

## Related tools

- **Snekmer** (Recodes proteins into AAR kmer vectors and runs unsupervised `cluster` mode, saving the pairwise distance matrix (save_matrix) and generating clustering figures (cluster_plots)) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Workflow engine underlying Snekmer; Snakemake arguments are passed through directly to Snakemake) — https://github.com/PNNL-CompBio/Snekmer
- **bsf-jaccard-py** (Blazing Signature Filter Python wrapper computing pairwise Jaccard similarity, used as the fast backend for agglomerative/density Jaccard clustering) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Python** (Binarizes the thresholded correlation matrix, computes connected components, and assembles per-threshold summaries (e.g. with pandas))

## Examples

```
snekmer cluster  # after configuring config.yaml with alphabet (e.g. solvacc, k=12) and save_matrix: True; verify install with `snekmer -h`
```

## Evaluation signals

- At correlation threshold 0.5 the pre-annotated nitrogen-cycle families form distinct, unconnected subnetworks with no edges between families (figure-5 behavior)
- The solvent accessibility alphabet with k=12 at threshold 0.1 produces clustering similar to the default network at reduced computational cost
- Every input protein is assigned to exactly one connected component of the thresholded graph
- The saved pairwise matrix is square with dimensions equal to the number of input proteins and is symmetric for a correlation/distance metric
- Network figures are produced when the cluster_plots option is enabled, matching Snekmer's documented clustering output

## Limitations

- Representing proteins as kmer sets uses an exact-match methodology that may be limited in terms of sensitivity and coverage; AAR mitigates but does not eliminate this for distantly related sequences
- The fast Jaccard clustering backend requires the BSF C extension (OpenMP 4.0, C++11; GCC 4.9+ recommended) or Snekmer falls back to scipy
- Threshold choice is a conservative trade-off: 0.5 separates nitrogen-cycle families but higher thresholds would be less permissive and lower thresholds risk merging families

## Evidence

- [intro] Six operation modes incl. cluster: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
- [methods] Agglomerative Jaccard default clustering: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [methods] min_rep k-mer occurrence filtering: "``min_rep``               ``int`` or ``None``    Discard k-mers with fewer than this many occurrences across the input set"
- [readme] AAR kmer representation and clustering objective: "Snekmer is a software package designed to reduce the representation of protein sequences
by combining amino acid reduction (AAR) with the kmer approach."
- [abstract] Exact-match kmer limitation: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [methods] save_matrix option documented: [evidence span withheld — anchor status: not_found]
