---
name: jaccard-distance-agglomerative-clustering
description: Use when protein sequences (e.g. a multi-genome UniProt/SwissProt FASTA
  set) must be grouped into families without labels, using amino-acid-reduced kmer
  sets rather than sequence alignment.
when_to_use_negative:
- When labeled training families are available and supervised classification is desired
  — use Snekmer's `model`/`apply` modes instead
- When sequences require alignment-based sensitivity (e.g. distant homologs needing
  MMseqs2 easy-cluster sensitivity rather than exact-match kmer overlap)
- When the input is already a feature table or distance matrix rather than raw protein
  sequences
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Snekmer
  role: Runs the agglomerative-jaccard clustering mode over AAR kmer vectors; default
    clustering method and dist_thresh configured via config.yaml
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: bsf-jaccard-py
  role: Blazing Signature Filter Python wrapper used to compute Jaccard similarity/distances
    at scale; Snekmer falls back to scipy if BSF is unavailable
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Snakemake
  role: Workflow engine through which the Snekmer cluster pipeline is executed
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:209b567382bf19cffeb7d7e721c8e8e376b820c3a19789ae5d67b61786cbf2a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Blazing Signature Filter
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# jaccard-distance-agglomerative-clustering

## Summary

Unsupervised clustering of proteins represented as AAR kmer vectors using agglomerative clustering on a Jaccard distance matrix, as implemented in the Snekmer `cluster` mode. This is the paper's method for de novo determination of protein families without requiring trained models.

## When to use

Use when protein sequences (e.g. a multi-genome UniProt/SwissProt FASTA set) must be grouped into families without labels, using amino-acid-reduced kmer sets rather than sequence alignment.

## When NOT to use

- When labeled training families are available and supervised classification is desired — use Snekmer's `model`/`apply` modes instead
- When sequences require alignment-based sensitivity (e.g. distant homologs needing MMseqs2 easy-cluster sensitivity rather than exact-match kmer overlap)
- When the input is already a feature table or distance matrix rather than raw protein sequences

## Inputs

- Protein sequences in FASTA format (e.g. a 10-genome SwissProt/UniProt dataset of 46,908 proteins)
- Snekmer config.yaml specifying alphabet reduction, k, linkage, and dist_thresh
- Optional annotation files (GO, Pfam, InterPro, TIGRFAMs, COG) for downstream V-score evaluation

## Outputs

- Cluster assignments for every input protein (agglomerative clusters over the Jaccard matrix)
- Jaccard distance matrix (BSF- or scipy-computed)
- Optional V-score (homogeneity/completeness harmonic mean) comparison table such as table3_vscore_comparison.csv

## How to apply

Recode each protein into an amino-acid-reduced (AAR) kmer vector via Snekmer's `cluster` mode (Snekmer 1.4.1, Python 3.11+, orchestrated by Snakemake), choosing an alphabet reduction and k (e.g. alphabet 0 hydrophobicity with k=14 performed best on the 10-genome benchmark). Build the pairwise Jaccard distance matrix using the Blazing Signature Filter (bsf-jaccard-py) for scalable computation, falling back to scipy.spatial.distance.pdist if BSF is unavailable. Run agglomerative clustering (the default Snekmer method) with complete linkage and a Jaccard dist_thresh of 95 to obtain cluster assignments. Benchmark quality by matching cluster assignments against UniProt annotations (GO, Pfam, InterPro, TIGRFAMs, COG) and computing V-scores (harmonic mean of homogeneity and completeness); optionally compare against MMseqs2 easy-cluster on the same input.

## Related tools

- **Snekmer** (Runs the agglomerative-jaccard clustering mode over AAR kmer vectors; default clustering method and dist_thresh configured via config.yaml) — https://github.com/PNNL-CompBio/Snekmer
- **bsf-jaccard-py** (Blazing Signature Filter Python wrapper used to compute Jaccard similarity/distances at scale; Snekmer falls back to scipy if BSF is unavailable) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Snakemake** (Workflow engine through which the Snekmer cluster pipeline is executed) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
snekmer cluster
```

## Evaluation signals

- A cluster assignment is produced for every protein in the input FASTA (e.g. all 46,908 proteins in the 10-genome benchmark)
- V-scores (harmonic mean of homogeneity and completeness) are computed per annotation source (GO, Pfam, InterPro, TIGRFAMs, COG) and reproduce the relative ordering of Table 3, with k=14 hydrophobicity AAR as the best Snekmer parameterization
- The Jaccard distance computation uses the specified alphabet reduction (e.g. alphabet 0 hydrophobicity, size 3->2) and k (e.g. 14); changing k or alphabet should change the clustering
- Clustering completes both with BSF installed and with the scipy.spatial.distance.pdist fallback, confirming pipeline robustness
- A side-by-side MMseqs2 easy-cluster V-score baseline is generated on the identical protein set for validation

## Limitations

- Representing proteins as kmer sets uses an exact-match methodology that may be limited in terms of sensitivity and coverage; AAR mitigates but does not remove this
- Snekmer V-scores were somewhat lower than MMseqs2 easy-cluster on the 10-genome benchmark, though still reasonable for annotation purposes
- BSF installation requires a C/C++ toolchain with OpenMP 4.0/C++11 (GCC 4.9+); otherwise Snekmer silently falls back to the slower scipy Jaccard path

## Evidence

- [readme] Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms: "Based on the AAR-kmer representations, Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
- [methods] Agglomerative clustering is the default Jaccard-based method with BSF or scipy fallback: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [readme] BSF enables efficient pairwise Jaccard similarity computation at scale: [evidence span withheld — anchor status: external_doc]
- [abstract] Clustering serves de novo protein family determination in Snekmer's design goals: "clustering for de novo determination of protein families"
