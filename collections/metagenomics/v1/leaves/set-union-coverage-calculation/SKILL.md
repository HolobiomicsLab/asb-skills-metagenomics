---
name: set-union-coverage-calculation
description: Use when two protein clustering or assignment methods have been run over
  the same protein universe and you need to measure non-shared fraction and union
  coverage gain from integration.
when_to_use_negative:
- Only one clustering method's assignments are available — set operations require
  two partitions over the same proteins
- The two method outputs cover different protein universes or use non-matching protein
  identifiers without prior harmonization
- You need cluster-level (rather than protein-membership-level) agreement metrics
  such as pairwise Jaccard between Snekmer clusters
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Snekmer
  role: Provides the AAR-kmer-based unsupervised clustering partition whose coverage
    is compared and unioned with MMseqs2 assignments
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: pandas-based set operations and coverage computation, writing cluster_overlap_comparison.csv
- name: bsf-jaccard-py
  role: Blazing Signature Filter Jaccard similarity backend used by Snekmer's clustering
    step that generates one input partition
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:881dde7d921f786ac51a16a38bdf524401f86f134d90575857e807c755915246
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Set-union coverage calculation

## Summary

Quantifies per-protein overlap between two clustering partitions (e.g., Snekmer AAR-kmer clusters and MMseqs2 sequence-similarity clusters) via intersection, symmetric difference, and union sets, and computes the coverage gain from integrating both methods — here raising assignment from ~50-60% per method to ~85% combined on the 10-genome Uniprot benchmark.

## When to use

Use when two protein clustering or assignment methods have been run over the same protein universe and you need to measure non-shared fraction and union coverage gain from integration.

## When NOT to use

- Only one clustering method's assignments are available — set operations require two partitions over the same proteins
- The two method outputs cover different protein universes or use non-matching protein identifiers without prior harmonization
- You need cluster-level (rather than protein-membership-level) agreement metrics such as pairwise Jaccard between Snekmer clusters

## Inputs

- Snekmer cluster assignment table for the 10-genome Uniprot benchmark (protein-to-cluster mapping)
- MMseqs2 cluster assignment table for the same 10-genome benchmark (protein-to-cluster mapping)
- Total protein universe / protein count for the benchmark set

## Outputs

- cluster_overlap_comparison.csv with shared/non-shared fractions, union coverage gain, per-method cluster counts, and mixed-cluster proportions
- Per-protein membership sets: intersection, symmetric difference, union between the two partitions

## How to apply

Load both cluster assignment tables (Snekmer cluster output and MMseqs2 cluster output for the 10-genome Uniprot benchmark) into pandas keyed on protein identifier. Build the set of proteins assigned by each method, then compute the intersection (shared), symmetric difference (non-shared), and union. Report per-method coverage as |assigned|/|total proteins| (~50-60% each for Snekmer and MMseqs2) and combined coverage as |union|/|total| (~85%). The rationale is that Snekmer's AAR-kmer representation links proteins with distant sequence similarity that exact sequence clustering misses, while MMseqs2 captures close homology Snekmer may miss, so the union is substantially larger than either partition alone; roughly 60% of clustered proteins are not shared between methods. Also count clusters per method and quantify mixed clusters (containing both annotated and unannotated proteins), writing all metrics to cluster_overlap_comparison.csv.

## Related tools

- **Snekmer** (Provides the AAR-kmer-based unsupervised clustering partition whose coverage is compared and unioned with MMseqs2 assignments) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (pandas-based set operations and coverage computation, writing cluster_overlap_comparison.csv)
- **bsf-jaccard-py** (Blazing Signature Filter Jaccard similarity backend used by Snekmer's clustering step that generates one input partition) — https://github.com/PNNL-CompBio/bsf-jaccard-py

## Examples

```
import pandas as pd; snek = pd.read_csv('snekmer_clusters.csv'); mm = pd.read_csv('mmseqs2_clusters.tsv', sep='\t'); union_cov = len(set(snek.protein) | set(mm.protein)) / total_proteins
```

## Evaluation signals

- Union coverage of the protein space reproduces the reported ~85% combined assignment figure
- Per-method coverage falls in the reported ~50-60% range for both Snekmer and MMseqs2
- Non-shared fraction of clustered proteins (symmetric difference / union of clustered sets) matches the reported ~60%
- Cluster counts per method and mixed-cluster proportions match Table 4 within rounding tolerance
- cluster_overlap_comparison.csv contains the overlap metrics, coverage gain, cluster counts, and mixed-cluster proportions

## Limitations

- Snekmer's kmer-set representation is exact-match based and may be limited in sensitivity and coverage; AAR mitigates but does not eliminate this
- Set-union coverage measures membership breadth, not cluster-quality agreement — clusters may overlap in membership yet partition proteins differently
- Findings are grounded in the 10-genome Uniprot benchmark; coverage fractions may not transfer to other protein sets

## Evidence

- [abstract] kmer-set sensitivity limitation: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [abstract] AAR links distant sequences: [evidence span withheld — anchor status: not_found]
- [intro] unsupervised clustering objective: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
- [methods] clustering algorithm options: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
