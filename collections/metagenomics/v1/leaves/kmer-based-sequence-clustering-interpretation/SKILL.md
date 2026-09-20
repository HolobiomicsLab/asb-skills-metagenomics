---
name: kmer-based-sequence-clustering-interpretation
description: Use when two clustering partitions (e.g., Snekmer AAR-kmer clusters and
  MMseqs2 sequence-similarity clusters) over the same protein set must be compared,
  and you need shared/non-shared fractions and union coverage (e.g., ~85% combined
  assignment).
when_to_use_negative:
- Only one clustering partition is available — set operations between two methods
  cannot be computed
- The goal is supervised family classification rather than de novo clustering overlap
  — use the Snekmer learn/apply pipeline and its snekmer_results.csv output instead
- Proteins have not yet been clustered by Snekmer (no AAR-kmer cluster assignments
  exist) — run Snekmer's cluster mode first
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Produces the AAR-kmer protein clusters (cluster mode) whose assignments are
    interpreted and compared against MMseqs2
  repo: http://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: pandas-based computation of set operations (intersection, symmetric difference,
    union), coverage fractions, and mixed-cluster proportions; export to CSV
- name: bsf-jaccard-py
  role: Jaccard similarity backend (Blazing Signature Filter) used by Snekmer clustering,
    with scipy fallback
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: MMseqs2
  role: Sequence-similarity clustering method whose cluster assignments serve as the
    comparison partition
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:ca4b783f77200b7436dbff16c70bc85fa3d444bef37cff81dc4662c088d47c22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMseqs2
  - Python
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# kmer-based-sequence-clustering-interpretation

## Summary

Interprets and compares protein clusters produced by kmer-based clustering (Snekmer, which recodes sequences into AAR kmer vectors) against sequence-similarity clustering (MMseqs2), quantifying overlap via per-protein set operations and combined coverage of the protein space.

## When to use

Use when two clustering partitions (e.g., Snekmer AAR-kmer clusters and MMseqs2 sequence-similarity clusters) over the same protein set must be compared, and you need shared/non-shared fractions and union coverage (e.g., ~85% combined assignment).

## When NOT to use

- Only one clustering partition is available — set operations between two methods cannot be computed
- The goal is supervised family classification rather than de novo clustering overlap — use the Snekmer learn/apply pipeline and its snekmer_results.csv output instead
- Proteins have not yet been clustered by Snekmer (no AAR-kmer cluster assignments exist) — run Snekmer's cluster mode first

## Inputs

- Snekmer cluster assignment table for the 10-genome Uniprot benchmark (protein-to-cluster mappings)
- MMseqs2 cluster assignment table (tab-separated protein-to-cluster mappings)
- Protein annotation table distinguishing annotated vs. unannotated proteins

## Outputs

- cluster_overlap_comparison.csv with overlap metrics, coverage gain, per-method cluster counts, and mixed-cluster proportions
- Per-protein intersection / symmetric difference / union sets of the two clusterings

## How to apply

Load the Snekmer cluster assignment table and the MMseqs2 cluster assignment table from the 10-genome Uniprot benchmark into pandas as per-protein mappings from protein ID to cluster ID. Compute per-protein set operations (intersection, symmetric difference, union) between the two partitions to derive the shared and non-shared fractions of clustered proteins; Snekmer relies on Jaccard distance over AAR-kmer sets (agglomerative clustering by default, via BSF with a scipy fallback), so its clusters capture distant similarity that exact sequence matching may miss. Compare single-method coverage (~50-60% of proteins each) against the union coverage (~85%). Count total clusters per method and compute the proportion of mixed clusters containing both annotated and unannotated proteins for each method, then write all metrics to cluster_overlap_comparison.csv. Validate by checking that the shared/unshared fraction reproduces the ~60% non-shared figure, union coverage reproduces ~85%, and cluster counts and mixed-cluster proportions match Table 4 within rounding tolerance.

## Related tools

- **Snekmer** (Produces the AAR-kmer protein clusters (cluster mode) whose assignments are interpreted and compared against MMseqs2) — http://github.com/PNNL-CompBio/Snekmer
- **Python** (pandas-based computation of set operations (intersection, symmetric difference, union), coverage fractions, and mixed-cluster proportions; export to CSV)
- **bsf-jaccard-py** (Jaccard similarity backend (Blazing Signature Filter) used by Snekmer clustering, with scipy fallback) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **MMseqs2** (Sequence-similarity clustering method whose cluster assignments serve as the comparison partition)

## Examples

```
import pandas as pd; snek = pd.read_csv('snekmer_clusters.csv'); mm = pd.read_csv('mmseqs2_clusters.tsv', sep='\t'); union_cov = len(set(snek.protein) | set(mm.protein)) / n_total_proteins
```

## Evaluation signals

- Union of proteins clustered by Snekmer and MMseqs2 equals ~85% of total proteins in the 10-genome benchmark
- Symmetric-difference fraction of clustered proteins reproduces the reported ~60% non-shared figure
- Per-method single coverage values fall in the ~50-60% range for both Snekmer and MMseqs2
- Total cluster counts and mixed-cluster (annotated + unannotated) proportions match Table 4 within rounding tolerance
- cluster_overlap_comparison.csv is produced with all overlap, coverage, and proportion columns populated

## Limitations

- Snekmer's kmer-set representation is an exact-match methodology that may be limited in sensitivity and coverage; AAR mitigates but does not eliminate this
- Low overlap between the two methods (~60% of clustered proteins not shared) means neither method's clusters alone can be treated as a superset of the other
- Agglomerative Jaccard clustering requires the BSF package; without it Snekmer falls back to scipy, which may affect scalability

## Evidence

- [intro] Snekmer reduces representation via AAR-kmer and clusters: "Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach."
- [methods] Agglomerative Jaccard clustering default: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [abstract] Exact-match limitation of kmer sets: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [readme] Six Snekmer operation modes including cluster: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
