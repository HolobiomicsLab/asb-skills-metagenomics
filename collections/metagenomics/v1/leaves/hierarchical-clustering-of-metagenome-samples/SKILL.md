---
name: hierarchical-clustering-of-metagenome-samples
description: Use when you have raw 16S rRNA or WGS metagenome reads for a sample set
  and need to cluster samples by community composition and abundance, or to compare
  how different distance metrics (Libra cosine, Mash MinHash, Simka) resolve sample
  groupings.
when_to_use_negative:
- Input is already a feature table or OTU/ASV abundance matrix rather than raw reads
  — Libra's k-mer counting pipeline expects FASTA/FASTQ sequence data
- Sample-to-sample comparison requires alignment-based taxonomic or functional profiling
  rather than composition-level k-mer clustering
- The dataset is too large for local processing and no Hadoop/CyVerse infrastructure
  is available (though the iMicrobe web implementation on CyVerse is an alternative)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Libra
  role: Computes the all-vs-all cosine similarity distance matrix on k-mer abundance,
    normalized for sequencing depth, that is then hierarchically clustered
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework underlying Libra's preprocessing and scoring
    runs (hadoop jar libra-all.jar)
- name: Mash
  role: Indexes each of the 48 samples by unique k-mers and computes the all-vs-all
    MinHash distance matrix for comparison of clustering resolution
- name: Simka
  role: Computes distance matrices between metagenomes using Jaccard-abundance and
    Bray-Curtis metrics as reduced-resolution baselines
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:f59a5171427b8f69fb20e06557c93d2a1fa8f1e36ba64dd4b9828813d00a7daf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hierarchical-clustering-of-metagenome-samples

## Summary

Cluster metagenomic samples hierarchically from all-vs-all distance matrices computed on k-mer content — e.g. Libra Cosine Similarity, Mash MinHash, and Simka Jaccard-abundance/Bray-Curtis matrices — to compare clustering resolution across distance metrics (Fig. 3 of the Libra paper, HMP 16S rRNA dataset).

## When to use

Use when you have raw 16S rRNA or WGS metagenome reads for a sample set and need to cluster samples by community composition and abundance, or to compare how different distance metrics (Libra cosine, Mash MinHash, Simka) resolve sample groupings.

## When NOT to use

- Input is already a feature table or OTU/ASV abundance matrix rather than raw reads — Libra's k-mer counting pipeline expects FASTA/FASTQ sequence data
- Sample-to-sample comparison requires alignment-based taxonomic or functional profiling rather than composition-level k-mer clustering
- The dataset is too large for local processing and no Hadoop/CyVerse infrastructure is available (though the iMicrobe web implementation on CyVerse is an alternative)

## Inputs

- FASTA/FASTQ metagenomic or amplicon read files per sample (e.g. 48 HMP 16S rRNA trimmed sequence reads)
- Libra k-mer index directory produced by the preprocess command
- All-vs-all distance matrices (Libra cosine similarity, Mash MinHash, Simka Jaccard-abundance, Simka Bray-Curtis)

## Outputs

- Libra all-vs-all score matrix written to the -o output directory
- Mash MinHash distance matrix across samples
- Simka Jaccard-abundance and Bray-Curtis distance matrices
- Hierarchical clusterings of the 48 samples per distance matrix, comparable to Fig. 3A–3D

## How to apply

Download the sample set (e.g. the 48 matched HMP 16S rRNA amplicon samples) as FASTA/FASTQ. For Libra, first build a k-mer index (preprocess, k-mer size k=20, NOTUNIQUE filter default), then run all-vs-all scoring with the COSINESIMILARITY scoring algorithm and LOGARITHM weighting in MAP run mode; Libra uses all reads and normalizes for sequencing depth. Compute comparison matrices with Mash (indexes each sample by unique k-mers, MinHash distances) and Simka (Jaccard-abundance and Bray-Curtis metrics). Cluster the 48 samples from each of the four distance matrices and compare the resulting hierarchical clusterings. Expect Mash to cluster by broad category at lower resolution than Libra, while both Simka presence/absence-style metrics fail to cluster by broad categories — evidence that data reduction and read count normalization diminish clustering resolution.

## Related tools

- **Libra** (Computes the all-vs-all cosine similarity distance matrix on k-mer abundance, normalized for sequencing depth, that is then hierarchically clustered) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework underlying Libra's preprocessing and scoring runs (hadoop jar libra-all.jar))
- **Mash** (Indexes each of the 48 samples by unique k-mers and computes the all-vs-all MinHash distance matrix for comparison of clustering resolution)
- **Simka** (Computes distance matrices between metagenomes using Jaccard-abundance and Bray-Curtis metrics as reduced-resolution baselines)

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir && hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Libra score matrix has 48×48 entries for the HMP sample set with unit self-similarity on the diagonal (cosine similarity of a sample with itself)
- Hierarchical clustering of the Libra cosine matrix groups samples by broad body-site category, matching Fig. 3B
- The Mash clustering reproduces the same broad categories but at visibly lower resolution than Libra (Fig. 3A vs 3B)
- Simka Jaccard-abundance and Bray-Curtis clusterings fail to group samples by broad categories, as reported in the paper
- Preprocess step completes without k-mer filter errors and the index directory contains one index file per file group per the -s/-g grouping options

## Limitations

- Presence/absence distance metrics and data reduction approaches (as used by Mash and Simka) greatly diminish clustering resolution compared with Libra's full-read abundance-based cosine similarity
- Mash indexes samples by unique k-mers, discarding abundance information that Libra retains
- Libra's default k-mer filter is NOTUNIQUE; alternative filters (NONE, STDDEV, STDDEV2) change index content and must be chosen deliberately

## Evidence

- [abstract] We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content.: "We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content."
- [results] Mash and Libra cluster samples by broad category with Mash at lower resolution; Simka metrics fail: [evidence span withheld — anchor status: not_found]
- [intro] The fastest of these methods, Mash, indexes samples by unique k-mers: "The fastest of these methods, Mash [15], indexes samples by unique k-mers"
- [intro] Simka computes a distance matrix between metagenomes: "Recently, Simka [15] was developed to compute a distance matrix between metagenomes"
- [readme] Scoring options from README: "- s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] README scoring invocation: [evidence span withheld — anchor status: external_doc]
