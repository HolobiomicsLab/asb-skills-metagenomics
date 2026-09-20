---
name: kmer-based-metagenome-distance-computation
description: Use when you have a set of FASTA/FASTQ metagenome samples (e.g. HMP 16S
  rRNA amplicons, HMP WGS reads, CAMI PacBio toy reads) and need an all-vs-all distance
  matrix to cluster samples by community composition and abundance.
when_to_use_negative:
- You only have a precomputed feature/abundance table or OTU matrix rather than raw
  FASTA/FASTQ reads — Libra computes k-mer abundance directly from read files
- A single-sample k-mer profile suffices and no pairwise comparison is needed — the
  skill targets all-vs-all matrices
- Compute cost must be minimized by sketching/sampling and resolution loss is acceptable
  — the paper shows such data reduction greatly diminishes resolution
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: 'Primary tool: preprocess FASTA/FASTQ into k-mer indexes, then compute the
    all-vs-all log-weighted Cosine Similarity score matrix via Hadoop MapReduce'
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework that Libra runs on (hadoop jar libra-all.jar)
    to index and score large sample sets
- name: Mash
  role: 'Benchmark: indexes samples by unique k-mers and computes MinHash distance
    matrices; clusters with lower resolution than Libra on HMP 16S and WGS data'
- name: Simka
  role: 'Benchmark: computes Jaccard-abundance and Bray-Curtis distance matrices;
    fails to cluster HMP samples by broad body-site categories'
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:8fe99da61e6be0b26e99e920ed3370c9c5f2dd94bd620854d4cea3348cb7132f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# kmer-based-metagenome-distance-computation

## Summary

Computes all-vs-all distance/similarity matrices between metagenomic samples from their k-mer content, as implemented by Libra (log-weighted Cosine Similarity on k-mer abundance in Hadoop), and benchmarks them against MinHash-based (Mash) and presence/absence-style (Simka Jaccard-ab, Bray-Curtis) metrics for sample clustering.

## When to use

Use when you have a set of FASTA/FASTQ metagenome samples (e.g. HMP 16S rRNA amplicons, HMP WGS reads, CAMI PacBio toy reads) and need an all-vs-all distance matrix to cluster samples by community composition and abundance.

## When NOT to use

- You only have a precomputed feature/abundance table or OTU matrix rather than raw FASTA/FASTQ reads — Libra computes k-mer abundance directly from read files
- A single-sample k-mer profile suffices and no pairwise comparison is needed — the skill targets all-vs-all matrices
- Compute cost must be minimized by sketching/sampling and resolution loss is acceptable — the paper shows such data reduction greatly diminishes resolution

## Inputs

- Metagenomic FASTA/FASTQ read files (e.g. 48 matched HMP 16S rRNA amplicon samples, 48 HMP WGS paired read files, 49 CAMI toy PacBio simulated samples)
- Hadoop index directories produced by Libra preprocess (e.g. /index_dir)

## Outputs

- All-vs-all similarity/distance matrices (e.g. Libra cosine similarity matrix in /score_dir, Mash MinHash distance matrix)
- Simka Jaccard-abundance and Bray-Curtis distance matrices
- Sample clusterings colored/labeled by body site for cross-tool comparison

## How to apply

Index each FASTA/FASTQ sample by k-mers with Libra's preprocess step (e.g. -k 20, NOTUNIQUE k-mer filter by default), then run Libra scoring (command 'core') with the default COSINESIMILARITY scoring algorithm, LOGARITHM weighting, and MAP run mode, writing the score matrix to -o /score_dir. For comparison, run Mash to index each sample by unique k-mers and emit a MinHash distance matrix, and run Simka to compute Jaccard-abundance and Bray-Curtis distance matrices. Cluster the samples from each matrix (e.g. hierarchical clustering) and label by known body site or origin. Rationale: Libra uses all reads to compute k-mer abundance while normalizing for sequencing depth, which the paper shows preserves clustering resolution that data reduction, read count normalization, and presence/absence metrics diminish.

## Related tools

- **Libra** (Primary tool: preprocess FASTA/FASTQ into k-mer indexes, then compute the all-vs-all log-weighted Cosine Similarity score matrix via Hadoop MapReduce) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework that Libra runs on (hadoop jar libra-all.jar) to index and score large sample sets)
- **Mash** (Benchmark: indexes samples by unique k-mers and computes MinHash distance matrices; clusters with lower resolution than Libra on HMP 16S and WGS data)
- **Simka** (Benchmark: computes Jaccard-abundance and Bray-Curtis distance matrices; fails to cluster HMP samples by broad body-site categories)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Scoring output directory (-o /score_dir) contains a complete, symmetric all-vs-all score matrix with one row/column per input sample (48 for HMP, 49 for CAMI toy)
- Clustering of the Libra cosine matrix reproduces the reported pattern: broad body-site clustering with higher resolution than Mash on both HMP 16S (Fig. 3A vs 3B) and HMP WGS (Fig. 4)
- Mash clusters should match Libra's broad categories but at decreased/lower resolution; divergence suggests an indexing or k-mer parameter error
- Simka Jaccard-ab and Bray-Curtis matrices on HMP data should show the known failure mode: skin and stool samples clustering with fornix samples
- On the CAMI toy PacBio dataset, all three tools should cluster samples broadly by body site with only small shared misclassifications attributable to PacBio error rate

## Limitations

- Presence/absence distance metrics, data reduction, and read count normalization greatly diminish clustering resolution of large-scale comparative analyses
- Mash consistently shows lower/decreased clustering resolution than Libra on HMP 16S and WGS data
- Simka's Jaccard-abundance and Bray-Curtis metrics miscluster HMP samples (some skin and stool samples cluster with fornix samples)
- Long-read technologies with high error rates (e.g. simulated PacBio) still produce small shared misclassifications across all tools
- Large-scale use requires a Hadoop deployment and CyVerse/iMicrobe infrastructure for the web-based route

## Evidence

- [abstract] Libra all-vs-all cosine on k-mer content: "We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content."
- [abstract] Cosine Similarity normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [abstract] Libra uses all reads at scale: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset"
- [readme] README scoring options: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] README weighting options: "w : weighting algorithm. LOGARITHM (default) | BOOLEAN | NATURAL"
- [results] Mash resolution finding from source cards: [evidence span withheld — anchor status: not_found]
