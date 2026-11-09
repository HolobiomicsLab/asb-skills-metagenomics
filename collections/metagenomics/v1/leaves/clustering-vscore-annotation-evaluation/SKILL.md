---
name: clustering-vscore-annotation-evaluation
description: Use when comparing de novo protein clusterings against reference functional
  annotations (GO, Pfam, InterPro, TIGRFAMs, COG) using V-scores, or when selecting
  AAR/kmer parameterizations by clustering quality.
when_to_use_negative:
- The task is supervised family classification rather than de novo clustering — use
  Snekmer's learn/apply (or easy) modes with family-specific thresholds instead.
- Input is already a cluster assignment or feature table; this skill requires raw
  protein sequences in FASTA to generate the clusterings being compared.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Snekmer
  role: Recodes proteins into AAR kmer vectors and runs agglomerative-jaccard unsupervised
    clustering for each parameterization being benchmarked
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Blazing Signature Filter
  role: Efficient pairwise Jaccard similarity computation backing the agglomerative-jaccard
    clustering (bsf-jaccard-py wrapper; scipy.spatial.distance.pdist fallback)
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Snakemake
  role: Workflow engine through which Snekmer orchestrates the clustering pipeline
  repo: https://github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:85d4da01fb4503082f1aabc81e9f48df60c547a0b8cde261784cf180fb5f8270
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Blazing Signature Filter
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# clustering-vscore-annotation-evaluation

## Summary

Benchmark unsupervised protein sequence clusterings (e.g., Snekmer AAR-kmer agglomerative-jaccard clustering) against a reference clustering such as MMseqs2 easy-cluster by scoring cluster assignments against UniProt annotations (GO, Pfam, InterPro, TIGRFAMs, COG) with homogeneity, completeness, and their harmonic mean (V-score).

## When to use

Use when comparing de novo protein clusterings against reference functional annotations (GO, Pfam, InterPro, TIGRFAMs, COG) using V-scores, or when selecting AAR/kmer parameterizations by clustering quality.

## When NOT to use

- The task is supervised family classification rather than de novo clustering — use Snekmer's learn/apply (or easy) modes with family-specific thresholds instead.
- Input is already a cluster assignment or feature table; this skill requires raw protein sequences in FASTA to generate the clusterings being compared.

## Inputs

- Protein FASTA file of the dataset to cluster (e.g., 46,908 SwissProt/UniProt proteins from 10 genomes per Supplementary Table S2)
- UniProt annotation files for GO, Pfam, InterPro, TIGRFAMs, and COG (e.g., .ann format like TIGRFAMs_annotation.ann)
- Snekmer config.yaml specifying AAR alphabet and k values, linkage, and dist_thresh
- MMseqs2 easy-cluster clustering results on the same dataset as reference

## Outputs

- Cluster assignments per protein for each Snekmer parameterization and for MMseqs2
- Homogeneity, completeness, and V-score per annotation source (GO, Pfam, InterPro, TIGRFAMs, COG) per clustering
- Comparison table CSV (e.g., table3_vscore_comparison.csv) with all four clustering results side by side

## How to apply

Cluster the input protein FASTA (e.g., the 46,908-protein 10-genome UniProt dataset) with 'snekmer cluster' using the agglomerative-jaccard method (Jaccard distance, complete linkage, dist_thresh 95) for each AAR/kmer parameterization (e.g., alphabet 5/miqs with k=4, alphabet 2/solvacc with k=8, alphabet 0/hydro with k=14). Run MMseqs2 easy-cluster with default sensitivity on the same FASTA as the reference clustering. Match each clustering's assignments against UniProt annotations from GO, Pfam, InterPro, TIGRFAMs, and COG and compute homogeneity, completeness, and the V-score (harmonic mean of homogeneity and completeness) per annotation source. Tabulate all V-scores (e.g., Snekmer parameterizations vs. MMseqs2) side by side as a CSV (table3_vscore_comparison.csv); Snekmer performed best with k=14 and the hydrophobicity AAR, achieving V-scores somewhat lower than MMseqs2 but still suitable for annotation purposes.

## Related tools

- **Snekmer** (Recodes proteins into AAR kmer vectors and runs agglomerative-jaccard unsupervised clustering for each parameterization being benchmarked) — https://github.com/PNNL-CompBio/Snekmer
- **Blazing Signature Filter** (Efficient pairwise Jaccard similarity computation backing the agglomerative-jaccard clustering (bsf-jaccard-py wrapper; scipy.spatial.distance.pdist fallback)) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Snakemake** (Workflow engine through which Snekmer orchestrates the clustering pipeline) — https://github.com/PNNL-CompBio/Snekmer

## Examples

```
snekmer cluster
```

## Evaluation signals

- The comparison table contains all four clustering results (three Snekmer parameterizations plus MMseqs2 easy-cluster) across all five annotation sources (GO, Pfam, InterPro, TIGRFAMs, COG).
- The relative V-score ordering reproduces the article's Table 3: best Snekmer performance with k=14 and the hydrophobicity (alphabet 0) AAR, with Snekmer V-scores somewhat lower than MMseqs2.
- Each reported V-score equals the harmonic mean of the separately computed homogeneity and completeness values for that annotation source.
- Every Snekmer clustering was run with complete linkage and dist_thresh 95 on the Jaccard matrix; verify these settings in the config/logs.
- The MMseqs2 baseline was run with default sensitivity on the identical 46,908-protein input so V-scores are directly comparable.

## Limitations

- Snekmer's kmer-set representation uses an exact-match methodology that may be limited in sensitivity and coverage relative to sequence-alignment tools like MMseqs2.
- The agglomerative-jaccard default clustering requires the BSF extension; if unavailable it falls back to scipy.spatial.distance.pdist, which may be slower at scale.
- AAR recoding trades exact kmer matching for linking distant sequence similarity, so V-scores are somewhat lower than MMseqs2, which the authors deemed 'still reasonably good for annotation purposes'.

## Evidence

- [methods] agglomerative-jaccard default clustering method: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [abstract] Snekmer clusters sequences using unsupervised algorithms on AAR-kmer representations: "Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models trained on input protein families or"
- [intro] six Snekmer operation modes including cluster: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
- [abstract] exact-match kmer limitation motivating V-score comparison: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
