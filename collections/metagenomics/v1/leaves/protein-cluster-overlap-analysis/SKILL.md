---
name: protein-cluster-overlap-analysis
description: Use when two protein clustering results (e.g., Snekmer cluster and MMseqs2
  outputs) on the same protein FASTA set must be compared via per-protein set operations,
  and combined coverage gain must be quantified.
when_to_use_negative:
- Input is a single clustering result with no second method's partition to compare
  against — the set operations require two partitions over the same proteins
- Protein identifier spaces of the two cluster tables do not match and cannot be reconciled,
  since per-protein set operations require a common key
- The goal is supervised family classification rather than unsupervised clustering
  comparison — use Snekmer's model/search (Learn/Apply) modes with cosine-similarity
  scores instead
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Produces one of the two cluster partitions via its unsupervised `cluster`
    mode on AAR-kmer representations of the 10-genome benchmark proteins
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runs the pandas-based set-operation analysis (intersection, symmetric difference,
    union) and writes cluster_overlap_comparison.csv
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the Snekmer clustering workflow whose outputs feed the overlap
    analysis
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:b6efb20b5836d13f1fd2dcabb37a3a13a9bda4fa19683f9f16cd5b2531413b74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# protein-cluster-overlap-analysis

## Summary

Quantifies the overlap between protein clusters produced by two independent clustering methods (Snekmer AAR-kmer clustering and MMseqs2) on a multi-genome protein set, and measures how much integrating both methods raises the fraction of proteins assigned to clusters.

## When to use

Use when two protein clustering results (e.g., Snekmer cluster and MMseqs2 outputs) on the same protein FASTA set must be compared via per-protein set operations, and combined coverage gain must be quantified.

## When NOT to use

- Input is a single clustering result with no second method's partition to compare against — the set operations require two partitions over the same proteins
- Protein identifier spaces of the two cluster tables do not match and cannot be reconciled, since per-protein set operations require a common key
- The goal is supervised family classification rather than unsupervised clustering comparison — use Snekmer's model/search (Learn/Apply) modes with cosine-similarity scores instead

## Inputs

- Snekmer cluster assignment table for the 10-genome Uniprot benchmark (protein-to-cluster mapping)
- MMseqs2 cluster assignment table for the same 10-genome protein set (protein-to-cluster mapping)
- Protein annotations (annotated vs. unannotated labels) used to flag mixed clusters

## Outputs

- cluster_overlap_comparison.csv with overlap metrics, coverage gain, per-method cluster counts, and mixed-cluster proportions
- Per-protein intersection / symmetric-difference / union membership of the two clusterings

## How to apply

Load the Snekmer cluster assignment table and the MMseqs2 cluster assignment table for the 10-genome Uniprot benchmark into pandas, keyed by protein identifier. Compute per-protein set operations — intersection, symmetric difference, and union — between the two partitions to derive the shared versus non-shared fraction of clustered proteins; the article reports each method clusters roughly 50-60% of proteins but ~60% of clustered proteins are not shared between methods. Compute single-method coverage versus the union coverage, which integrating both methods raises to ~85% of total proteins. Count total clusters per method and quantify the proportion of mixed clusters containing both annotated and unannotated proteins (reported in Table 4). Write overlap metrics, coverage gain, cluster counts, and mixed-cluster proportions to cluster_overlap_comparison.csv. Validation: shared/unshared fractions and the union-coverage value must reproduce the reported ~60% non-shared and ~85% combined figures, and cluster counts and mixed-cluster proportions must match Table 4 within rounding tolerance.

## Related tools

- **Snekmer** (Produces one of the two cluster partitions via its unsupervised `cluster` mode on AAR-kmer representations of the 10-genome benchmark proteins) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Runs the pandas-based set-operation analysis (intersection, symmetric difference, union) and writes cluster_overlap_comparison.csv) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the Snekmer clustering workflow whose outputs feed the overlap analysis) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
import pandas as pd; snek = pd.read_csv('snekmer_clusters.tsv', sep='\t', index_col=0); mm = pd.read_csv('mmseqs2_clusters.tsv', sep='\t', index_col=0); shared = snek.index.intersection(mm.index); union_cov = len(snek.index.union(mm.index)) / n_total
```

## Evaluation signals

- Computed shared vs. non-shared fraction of clustered proteins reproduces the reported ~60% non-shared figure
- Union of the two clusterings assigns ~85% of total proteins to clusters (vs. ~50-60% per single method)
- Per-method total cluster counts match Table 4 within rounding tolerance
- Proportion of mixed clusters (containing both annotated and unannotated proteins) matches Table 4 within rounding tolerance
- cluster_overlap_comparison.csv is written and contains the overlap metrics, coverage gain, cluster counts, and mixed-cluster proportions

## Limitations

- Snekmer's kmer-set representation uses an exact-match methodology that may be limited in sensitivity and coverage, which propagates into the overlap statistics
- The reported figures (~60% non-shared, ~85% combined coverage) are specific to the 10-genome Uniprot benchmark and will not transfer unchanged to other protein sets
- Mixed-cluster proportions depend on annotation availability; unannotated proteins can only be flagged if annotations are provided as input

## Evidence

- [intro] Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms, (2) generates supervised machine learning models: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms, (2) generates supervised machine learning models"
- [abstract] this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [abstract] Snekmer is written in Python using Snakemake.: "Snekmer is written in Python using Snakemake."
- [abstract] Code and data used in this article, along with tutorial notebooks, are available at http://github.com/PNNL-CompBio/Snekmer under an open-source BSD-3 license.: "Code and data used in this article, along with tutorial notebooks, are available at http://github.com/PNNL-CompBio/Snekmer"
