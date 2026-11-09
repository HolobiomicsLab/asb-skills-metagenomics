---
name: clustering-method-benchmark-comparison
description: Use when two protein clustering methods (e.g. Snekmer AAR-kmer clustering
  vs MMseqs2 easy-cluster) have been run on the same protein FASTA set and you must
  benchmark them via V-scores and quantify overlap/union coverage of assigned proteins.
when_to_use_negative:
- The goal is supervised family prediction with per-sequence confidence — use Snekmer's
  learn/apply mode, which outputs results/apply/snekmer_results.csv with Prediction,
  Score, delta, and Confidence columns instead of cluster partitions.
- Only one clustering method's assignments are available — both the V-score comparison
  and the overlap/union coverage analysis require two partitions over the same protein
  set.
- Input sequences are not proteins (e.g. nucleotide reads) — AAR kmer recoding and
  the UniProt annotation benchmark assume protein FASTA input.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: Snekmer
  role: Runs 'snekmer cluster' (agglomerative-jaccard, complete linkage, dist_thresh
    95) for each AAR alphabet/kmer parameterization to produce cluster assignment
    tables for benchmarking
  repo: http://github.com/PNNL-CompBio/Snekmer
- name: MMseqs2
  role: Baseline sequence-similarity clusterer, run as easy-cluster with default sensitivity
    on the same 46,908-protein dataset to provide the reference V-score and partition
    for overlap analysis
- name: Blazing Signature Filter (bsf-jaccard-py)
  role: Computes the Jaccard similarity matrix consumed by Snekmer's agglomerative
    clustering; Snekmer falls back to scipy.spatial.distance.pdist when BSF is unavailable
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
- name: Python (pandas)
  role: Loads the two cluster assignment tables and computes intersection/symmetric-difference/union
    set operations, coverage fractions, cluster counts, and mixed-cluster proportions
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:ac0fcb694308345ac55b2961a77ea24ffe6c5d3455b65a885adb18e5c0fbb1b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Blazing Signature Filter
  - Blazing Signature Filter (bsf-jaccard-py)
  - MMseqs2
  - Python
  - Python (pandas)
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# clustering-method-benchmark-comparison

## Summary

Benchmarks unsupervised protein clustering methods — Snekmer AAR-kmer agglomerative-jaccard clustering versus MMseqs2 easy-cluster — on the same dataset using V-scores (harmonic mean of homogeneity and completeness) over functional annotations, and quantifies whether integrating both partitions increases protein-cluster coverage (here from ~50-60% per method to 85%).

## When to use

Use when two protein clustering methods (e.g. Snekmer AAR-kmer clustering vs MMseqs2 easy-cluster) have been run on the same protein FASTA set and you must benchmark them via V-scores and quantify overlap/union coverage of assigned proteins.

## When NOT to use

- The goal is supervised family prediction with per-sequence confidence — use Snekmer's learn/apply mode, which outputs results/apply/snekmer_results.csv with Prediction, Score, delta, and Confidence columns instead of cluster partitions.
- Only one clustering method's assignments are available — both the V-score comparison and the overlap/union coverage analysis require two partitions over the same protein set.
- Input sequences are not proteins (e.g. nucleotide reads) — AAR kmer recoding and the UniProt annotation benchmark assume protein FASTA input.

## Inputs

- 10-genome SwissProt/UniProt protein FASTA dataset (46,908 proteins; genome list per Supplementary Table S2)
- Snekmer cluster assignment tables for three parameterizations (alphabet 5/k=4, alphabet 2/k=8, alphabet 0/k=14)
- MMseqs2 easy-cluster assignment table for the same FASTA
- UniProt annotation files (GO, Pfam, InterPro, TIGRFAMs, COG)

## Outputs

- table3_vscore_comparison.csv — V-scores for the three Snekmer parameterizations and MMseqs2 per annotation source
- cluster_overlap_comparison.csv — shared/non-shared fractions, single-method vs union coverage, cluster counts, mixed-cluster proportions
- Best-parameterization determination (k=14, hydrophobicity AAR)

## How to apply

Run each Snekmer parameterization via 'snekmer cluster' with the agglomerative-jaccard method (complete linkage, dist_thresh 95 on the Jaccard matrix): alphabet 5 (miqs) k=4, alphabet 2 (solvacc) k=8, and alphabet 0 (hydrophobicity) k=14, then run MMseqs2 easy-cluster with default sensitivity on the same 46,908-protein 10-genome UniProt FASTA. Match cluster assignments against UniProt annotations (GO, Pfam, InterPro, TIGRFAMs, COG) and compute a V-score per annotation source for all four clustering results. Load both cluster assignment tables into pandas and compute per-protein set operations (intersection, symmetric difference, union) to split clustered proteins into shared vs non-shared fractions and derive single-method (~50-60%) versus union (~85%) coverage of the protein space. Write V-scores to table3_vscore_comparison.csv and overlap metrics, coverage gain, per-method cluster counts, and mixed-cluster (annotated+unannotated) proportions to cluster_overlap_comparison.csv, validating against Tables 3 and 4.

## Related tools

- **Snekmer** (Runs 'snekmer cluster' (agglomerative-jaccard, complete linkage, dist_thresh 95) for each AAR alphabet/kmer parameterization to produce cluster assignment tables for benchmarking) — http://github.com/PNNL-CompBio/Snekmer
- **MMseqs2** (Baseline sequence-similarity clusterer, run as easy-cluster with default sensitivity on the same 46,908-protein dataset to provide the reference V-score and partition for overlap analysis)
- **Blazing Signature Filter (bsf-jaccard-py)** (Computes the Jaccard similarity matrix consumed by Snekmer's agglomerative clustering; Snekmer falls back to scipy.spatial.distance.pdist when BSF is unavailable) — https://github.com/PNNL-CompBio/bsf-jaccard-py
- **Python (pandas)** (Loads the two cluster assignment tables and computes intersection/symmetric-difference/union set operations, coverage fractions, cluster counts, and mixed-cluster proportions)

## Examples

```
snekmer cluster  # parameterization 3: alphabet 0 (hydrophobicity AAR), k=14, agglomerative-jaccard, complete linkage, dist_thresh 95
```

## Evaluation signals

- table3_vscore_comparison.csv contains all four clustering results (three Snekmer parameterizations plus MMseqs2) and reproduces the relative V-score values/ordering reported in Table 3 for each annotation source (GO, Pfam, InterPro, TIGRFAMs, COG).
- cluster_overlap_comparison.csv reproduces the reported ~60% non-shared fraction among clustered proteins and ~85% combined union coverage of the 46,908-protein space.
- Single-method coverage values each fall in the ~50-60% range before integration.
- Per-method cluster counts and mixed-cluster (annotated+unannotated) proportions match Table 4 within rounding tolerance.
- The best Snekmer parameterization is identified as k=14 with the hydrophobicity AAR (alphabet 0), consistent with the reported ordering.

## Limitations

- Snekmer's V-scores are somewhat lower than MMseqs2's on the 10-genome UniProt benchmark (Table 3), though still considered reasonably good for annotation purposes.
- The kmer representation is an exact-match methodology that may be limited in sensitivity and coverage; amino acid recoding (AAR) is the mitigating strategy, so results depend on alphabet/k choice.
- Agglomerative-jaccard clustering requires the compiled BSF extension (bsf-jaccard-py, needs GCC 4.9+ / OpenMP); without it Snekmer falls back to scipy.spatial.distance.pdist, which may affect runtime and numeric behavior.
- Coverage figures (~50-60% per method, 85% union) are specific to the 46,908-protein 10-genome UniProt dataset and dist_thresh 95 clustering settings.

## Evidence

- [abstract] Snekmer accurately differentiates nitrogen cycling families and Uniprot proteins in both supervised (model) and unsupervised (clustering) operation modes.: [evidence span withheld — anchor status: not_found]
- [abstract] Exact-match kmer limitation motivating AAR recoding and careful parameterization comparison: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [methods] Default agglomerative-jaccard clustering method used for the Snekmer side of the benchmark: "Agglomerative clustering using Jaccard distance (default; requires BSF or falls back to scipy)"
- [readme] Snekmer's clustering mode is the operation under benchmark: "Snekmer subsequently (1) clusters sequences using various unsupervised clustering algorithms,"
- [results] Distinguishes this clustering skill from Snekmer's supervised learn/apply output: "The main output is a CSV file at ``results/apply/snekmer_results.csv``"
