---
name: minhash-genome-sketching
description: Use when you have paired WGS/FASTQ read files for many metagenomic samples
  and need a fast MinHash-based all-vs-all distance matrix to test whether samples
  cluster by a known metadata attribute such as body site.
when_to_use_negative:
- When abundance resolution matters — the article reports Mash's data-reduction/unique-k-mer
  approach gives decreased clustering resolution versus Libra's all-read k-mer abundance
  calculation
- When inputs are already assembled genomes or sketch files rather than raw WGS reads
  for all-vs-all metagenome comparison
- When the target distance requires abundance-aware normalization (e.g., Cosine Similarity
  on composition and abundance normalized for sequencing depth) rather than MinHash
  presence/absence-style distance
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Mash
  role: Sketches each HMP WGS read sample and computes the all-vs-all MinHash distance
    matrix used for body-site clustering in the Fig. 4 benchmark
- name: Libra
  role: Benchmarked alongside Mash in MAP mode with COSINESIMILARITY scoring and LOGARITHM
    weighting to produce a comparative k-mer abundance score matrix
  repo: iychoi/libra
- name: Simka
  role: Computes Jaccard-ab and Bray-Curtis distance matrices on the same read set
    as a comparison metric that fails to cluster HMP samples by body site
- name: Hadoop
  role: Underlying scalable framework Libra uses for the all-vs-all metagenome comparison
    in this benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:ce5c6345be4b747c9af938c162e4b50c00592a914ec6ea886b2cc847916973eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# minhash-genome-sketching

## Summary

Sketches metagenomic samples with Mash MinHash and computes an all-vs-all distance matrix to cluster samples (e.g., 48 HMP whole-genome shotgun read samples by body site), benchmarking Mash's resolution against abundance-aware tools like Libra and Simka.

## When to use

Use when you have paired WGS/FASTQ read files for many metagenomic samples and need a fast MinHash-based all-vs-all distance matrix to test whether samples cluster by a known metadata attribute such as body site.

## When NOT to use

- When abundance resolution matters — the article reports Mash's data-reduction/unique-k-mer approach gives decreased clustering resolution versus Libra's all-read k-mer abundance calculation
- When inputs are already assembled genomes or sketch files rather than raw WGS reads for all-vs-all metagenome comparison
- When the target distance requires abundance-aware normalization (e.g., Cosine Similarity on composition and abundance normalized for sequencing depth) rather than MinHash presence/absence-style distance

## Inputs

- 48 paired whole-genome shotgun (WGS) read files from the Human Microbiome Project (HMIWGS) dataset
- Sample metadata (body site labels for the 48 HMP samples) for clustering evaluation

## Outputs

- Mash MinHash sketches per sample
- All-vs-all Mash MinHash distance matrix
- Comparative Libra COSINESIMILARITY score matrix (score_dir) and Simka Jaccard-ab / Bray-Curtis matrices
- Body-site clustering assignments for the 48 samples under each distance metric

## How to apply

Obtain the paired whole-genome shotgun read files (e.g., the 48-sample HMIWGS HMP dataset), then run Mash to sketch each sample and compute the all-vs-all MinHash distance matrix. In the Libra paper benchmark (Fig. 4), the same 48 HMP WGS read samples are also processed with Libra in MAP mode using the default COSINESIMILARITY scoring and LOGARITHM weighting (`m=MAP`, `s` default, `w` default, output written to a score directory via `-o`), and with Simka to compute Jaccard-ab and Bray-Curtis matrices. Cluster the 48 samples by body site for each distance matrix and compare outcomes. The expected pattern is that Mash and Libra both enhance body-site clustering, but Mash shows decreased resolution compared to Libra because Mash indexes samples by unique k-mers, while Simka's Jaccard-ab and Bray-Curtis metrics fail to cluster by body site, with some skin and stool samples misclustered with fornix samples. This reduced-resolution behavior is attributed to data-reduction methods such as sketching/unique-k-mer indexing and presence/absence metrics, which the article argues greatly diminish the resolution of large-scale comparative analyses.

## Related tools

- **Mash** (Sketches each HMP WGS read sample and computes the all-vs-all MinHash distance matrix used for body-site clustering in the Fig. 4 benchmark)
- **Libra** (Benchmarked alongside Mash in MAP mode with COSINESIMILARITY scoring and LOGARITHM weighting to produce a comparative k-mer abundance score matrix) — iychoi/libra
- **Simka** (Computes Jaccard-ab and Bray-Curtis distance matrices on the same read set as a comparison metric that fails to cluster HMP samples by body site)
- **Hadoop** (Underlying scalable framework Libra uses for the all-vs-all metagenome comparison in this benchmark)

## Examples

```
hadoop jar libra-all.jar core -m MAP -s COSINESIMILARITY -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Mash distance matrix is all-vs-all over 48 HMP WGS samples with no missing pairs
- Clustering by body site is enhanced for Mash and Libra, with Mash at decreased resolution compared to Libra
- Both Simka Jaccard-ab and Bray-Curtis matrices show samples remaining misclustered by body site, with some skin and stool samples clustering with fornix samples
- Libra score matrix was produced in MAP mode with default COSINESIMILARITY scoring and LOGARITHM weighting written to the specified score_dir

## Limitations

- Mash shows decreased clustering resolution compared to Libra on HMP WGS reads because it indexes samples by unique k-mers (data reduction / presence/absence-style behavior)
- Simka's Jaccard-ab and Bray-Curtis metrics fail to cluster the 48 HMP WGS read samples by body site
- Data reduction, read count normalization, and presence/absence distance metrics generally greatly diminish the resolution of large-scale comparative analyses

## Evidence

- [results] Mash decreased resolution vs Libra on HMP WGS reads: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution"
- [intro] Mash indexes samples by unique k-mers: "The fastest of these methods, Mash [15], indexes samples by unique k-mers"
- [abstract] Libra uses all reads for k-mer abundance in scalable Hadoop architecture: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset"
- [readme] README scoring command with weighting and output options: [evidence span withheld — anchor status: external_doc]
- [readme] README scoring algorithm options: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [intro] HMP 48-sample dataset used in the study: "48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project (HMP)"
