---
name: log-weighted-cosine-similarity-scoring
description: Use when you have FASTA/FASTQ metagenome samples (short- or long-read,
  e.g. simulated PacBio) already indexed by Libra and need a pairwise distance/similarity
  matrix that accounts for both k-mer composition and abundance for clustering.
when_to_use_negative:
- Inputs are already a feature/abundance table or an existing distance matrix rather
  than raw FASTA/FASTQ reads to index and score
- You specifically need a presence/absence (composition-only) comparison or want read-count
  normalization explicitly avoided — Libra's design normalizes for depth via Cosine
  Similarity
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Performs the all-vs-all k-mer-based comparison; the HM/core scoring command
    with COSINESIMILARITY and LOGARITHM options produces the distance matrix
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework Libra runs on (hadoop jar libra-all.jar) for
    massive metagenome comparisons
- name: Mash
  role: Alternative MinHash-based tool used in the CAMI benchmark to emit a pairwise
    distance matrix for cross-tool comparison
- name: Simka
  role: Alternative tool computing Jaccard-ab and Bray-Curtis distance matrices over
    the same samples for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:aff210c95336b841729cb8e0dee927f6a6d859e5e62a5e55594c0dba09ae144e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# log-weighted-cosine-similarity-scoring

## Summary

Libra's HM scoring command computes an all-vs-all metagenome distance matrix using Cosine Similarity over k-mer abundance vectors, with LOGARITHM weighting to down-weight high-abundance k-mers, normalizing for sequencing depth while retaining all reads.

## When to use

Use when you have FASTA/FASTQ metagenome samples (short- or long-read, e.g. simulated PacBio) already indexed by Libra and need a pairwise distance/similarity matrix that accounts for both k-mer composition and abundance for clustering.

## When NOT to use

- Inputs are already a feature/abundance table or an existing distance matrix rather than raw FASTA/FASTQ reads to index and score
- You specifically need a presence/absence (composition-only) comparison or want read-count normalization explicitly avoided — Libra's design normalizes for depth via Cosine Similarity

## Inputs

- Libra k-mer index directory (/index_dir) produced by the preprocess step from FASTA/FASTQ metagenome reads
- Metagenomic samples such as the 49-sample CAMI toy PacBio dataset or 48 HMP 16S/WGS samples

## Outputs

- Pairwise cosine-similarity distance matrix written to /score_dir (one score per sample pair, log-weighted)
- Sample clustering (e.g. hierarchical) colored/labelled by body site or other metadata

## How to apply

First run Libra's preprocessing step (hadoop jar libra-all.jar preprocess -k 20) on the FASTA/FASTQ sample directory to build the /index_dir of k-mer indexes. Then run the scoring command with the default COSINESIMILARITY scoring algorithm and LOGARITHM weighting in MAP mode ('core -w LOGARITHM') to emit the log-weighted cosine distance matrix into /score_dir. Choose COSINESIMILARITY when you want genetic distance and microbial abundance considered simultaneously; alternatives are BRAYCURTIS or JENGENSHANNON-type JENSENSHANNON scoring. Rationale: Libra uses all reads to calculate k-mer abundance in a Hadoop architecture, and Cosine Similarity normalizes for sequencing depth, so data reduction and presence/absence metrics do not diminish resolution. Finally, feed the distance matrix to hierarchical clustering and label samples by known metadata (e.g. HMP body site) to validate the grouping.

## Related tools

- **Libra** (Performs the all-vs-all k-mer-based comparison; the HM/core scoring command with COSINESIMILARITY and LOGARITHM options produces the distance matrix) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework Libra runs on (hadoop jar libra-all.jar) for massive metagenome comparisons)
- **Mash** (Alternative MinHash-based tool used in the CAMI benchmark to emit a pairwise distance matrix for cross-tool comparison)
- **Simka** (Alternative tool computing Jaccard-ab and Bray-Curtis distance matrices over the same samples for comparison)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Score output directory /score_dir contains a complete all-vs-all matrix (49x49 for the CAMI HMP toy PacBio set) with no missing sample pairs
- Hierarchical clustering of the Libra distance matrix places samples broadly by known body-site origin, matching Mash and Simka results on the same dataset
- Any misclassifications are small and shared across Mash, Libra, and Simka, attributable to PacBio error rate rather than the scoring metric
- Scoring options match the validated configuration: COSINESIMILARITY (default) algorithm, LOGARITHM weighting, MAP mode

## Limitations

- Increased PacBio read error rate caused small misclassifications shared by all tools on the CAMI toy dataset
- Common compute-time shortcuts — data reduction, read count normalization, presence/absence distance metrics — greatly diminish resolution; Libra is designed to avoid these but therefore processes all reads
- Scoring is limited to the three implemented algorithms (COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON); other metrics are not available

## Evidence

- [abstract] Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [intro] cosine similarity rationale: "We demonstrate that Cosine Similarity, which is widely used in document clustering and information retrieval, is a good distance metric for comparing datasets to consider genetic distance and"
- [methods] scoring options: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] scoring command: [evidence span withheld — anchor status: external_doc]
- [readme] weighting options: "w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
