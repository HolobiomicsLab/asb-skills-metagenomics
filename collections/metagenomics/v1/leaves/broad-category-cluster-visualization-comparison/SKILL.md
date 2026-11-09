---
name: broad-category-cluster-visualization-comparison
description: Use when evaluating whether a k-mer MinHash tool's clustering resolution
  on 16S amplicon metagenomes is adequate by comparing its sample-cluster visualization
  against a full k-mer abundance cosine-similarity method.
when_to_use_negative:
- Inputs are whole-genome shotgun (WGS) HMP samples or the CAMI toy PacBio dataset
  — reproduce the corresponding Fig. 2/4 workflows instead of the 16S amplicon comparison
- The goal is to test a novel scoring algorithm (e.g. BRAYCURTIS or JENSENSHANNON
  in Libra) rather than the default COSINESIMILARITY comparison against Mash
- Samples number in the thousands/millions and a small 48-sample visual check is not
  the intended deliverable
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Libra
  role: computes the all-vs-all cosine-similarity k-mer abundance score matrix (s=COSINESIMILARITY,
    w=LOGARITHM, m=MAP) on the 48 HMP 16S samples
  repo: https://github.com/iychoi/libra
- name: Mash
  role: indexes each of the 48 samples by unique k-mers and computes the all-vs-all
    MinHash distance matrix for the lower-resolution clustering comparison
- name: Simka
  role: computes distance matrices between the metagenomes using both Jaccard-abundance
    and Bray-Curtis metrics as the negative-control clustering result
- name: Hadoop
  role: execution framework under which Libra's MapReduce all-vs-all scoring is run
    (hadoop jar libra-all.jar core)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:4326473ad818463c2be4d6f76688ef04a12fca0a9867f450e0268c204c5516b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# broad-category-cluster-visualization-comparison

## Summary

Reproduces the Libra paper's Fig. 3 comparison of all-vs-all distance matrices (Mash MinHash, Libra cosine similarity, Simka Jaccard-abundance, Simka Bray-Curtis) on 48 matched HMP 16S rRNA amplicon samples, showing that Mash clusters at lower resolution than Libra.

## When to use

Use when evaluating whether a k-mer MinHash tool's clustering resolution on 16S amplicon metagenomes is adequate by comparing its sample-cluster visualization against a full k-mer abundance cosine-similarity method.

## When NOT to use

- Inputs are whole-genome shotgun (WGS) HMP samples or the CAMI toy PacBio dataset — reproduce the corresponding Fig. 2/4 workflows instead of the 16S amplicon comparison
- The goal is to test a novel scoring algorithm (e.g. BRAYCURTIS or JENSENSHANNON in Libra) rather than the default COSINESIMILARITY comparison against Mash
- Samples number in the thousands/millions and a small 48-sample visual check is not the intended deliverable

## Inputs

- 48 matched Human Microbiome Project (HMP) 16S rRNA trimmed amplicon sequence read FASTA/FASTQ files
- Pre-built Libra binary (libra-all.jar, e.g. v1.2.8) and a Hadoop installation
- Mash and Simka installations for the comparison distance matrices

## Outputs

- Libra all-vs-all cosine similarity score matrix over the 48 samples (written to the -o score directory)
- Mash all-vs-all MinHash distance matrix indexed by unique k-mers
- Simka Jaccard-abundance and Bray-Curtis distance matrices
- Cluster diagrams of the 48 samples from each of the four distance matrices, comparable to Fig. 3A–3D of the paper

## How to apply

Download the 48 matched HMP 16S rRNA amplicon samples from the NIH Human Microbiome Project 16S trimmed sequence read dataset. Run Libra in all-vs-all scoring mode (hadoop jar libra-all.jar core) with default scoring algorithm s=COSINESIMILARITY, weighting w=LOGARITHM, and run mode m=MAP, writing results to the -o directory. Separately run Mash to index each of the 48 samples by unique k-mers and compute the all-vs-all MinHash distance matrix, and run Simka to compute both Jaccard-abundance and Bray-Curtis distance matrices. Cluster all 48 samples from each of the four distance matrices and compare the resulting cluster diagrams against the published Fig. 3, confirming the expected pattern: Mash and Libra both cluster samples by broad category, but Mash does so at lower resolution, while both Simka metrics fail to cluster by broad categories. The resolution gap arises because Mash indexes only unique k-mers (data reduction and presence/absence metrics) whereas Libra uses all reads to calculate k-mer abundance, normalized for sequencing depth.

## Related tools

- **Libra** (computes the all-vs-all cosine-similarity k-mer abundance score matrix (s=COSINESIMILARITY, w=LOGARITHM, m=MAP) on the 48 HMP 16S samples) — https://github.com/iychoi/libra
- **Mash** (indexes each of the 48 samples by unique k-mers and computes the all-vs-all MinHash distance matrix for the lower-resolution clustering comparison)
- **Simka** (computes distance matrices between the metagenomes using both Jaccard-abundance and Bray-Curtis metrics as the negative-control clustering result)
- **Hadoop** (execution framework under which Libra's MapReduce all-vs-all scoring is run (hadoop jar libra-all.jar core))

## Examples

```
hadoop jar libra-all.jar core -s COSINESIMILARITY -w LOGARITHM -m MAP -o /score_dir /index_dir
```

## Evaluation signals

- Libra and Mash cluster diagrams both group the 48 samples by broad category, but the Mash diagram resolves fewer clusters (Mash lower resolution than Libra, per Fig. 3A vs 3B)
- Both Simka Jaccard-abundance and Simka Bray-Curtis cluster diagrams fail to group the samples by broad category
- Libra scoring was run with the documented defaults: s=COSINESIMILARITY, w=LOGARITHM, m=MAP
- The Libra score matrix contains all 48×47 (ordered) pairwise scores, one set per sample pair, in the -o output directory
- Mash matrix is derived from unique-k-mer indexes of all 48 samples; Simka matrices include both Jaccard-abundance and Bray-Curtis variants

## Limitations

- Results depend on the preprocessing index parameters (k-mer size via preprocess -k, and the NOTUNIQUE default kmer filter -f); changing these alters the comparison
- Data reduction, read count normalization, and presence/absence distance metrics (as used by Mash and Simka) greatly diminish clustering resolution compared with Libra's full-read k-mer abundance cosine similarity
- Comparison is qualitative — cluster diagrams are validated visually against the published Fig. 3, not by a numeric correspondence metric

## Evidence

- [results] Mash lower resolution than Libra on HMP 16S: [evidence span withheld — anchor status: not_found]
- [abstract] Data reduction and presence/absence metrics reduce resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [abstract] Libra uses all reads for k-mer abundance: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [intro] HMP 48-sample dataset: "48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project (HMP)"
- [intro] Mash indexes samples by unique k-mers: "The fastest of these methods, Mash [15], indexes samples by unique k-mers"
- [methods] Libra scoring options: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] README core scoring command: [evidence span withheld — anchor status: external_doc]
