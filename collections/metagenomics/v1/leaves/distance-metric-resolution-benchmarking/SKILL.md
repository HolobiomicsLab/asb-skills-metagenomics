---
name: distance-metric-resolution-benchmarking
description: Use when choosing among metagenome comparison tools/metrics and you need
  evidence on which metric preserves body-site or community-composition structure
  rather than collapsing resolution.
when_to_use_negative:
- When the goal is a single tool's output quality rather than a head-to-head metric
  comparison — use the tool's own validation workflow instead
- When samples lack a known grouping (e.g., body site) against which clustering resolution
  can be judged; benchmarking resolution requires a ground-truth label
- When the input is not raw reads (e.g., already-computed distance matrices or sketch
  files) — re-running Mash/Libra/Simka on derived objects would not reproduce the
  read-level abundance comparison
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Computes all-vs-all k-mer abundance scores (COSINESIMILARITY by default) on
    raw reads, run in MAP mode with LOGARITHM weighting; serves as the abundance-preserving
    metric under test
  repo: https://github.com/iychoi/libra
- name: Mash
  role: Sketches each sample and computes the all-vs-all MinHash distance matrix;
    baseline data-reduction metric compared against Libra
- name: Simka
  role: Computes Jaccard-ab and Bray-Curtis distance matrices between read sets; the
    presence/absence-style metrics that fail to cluster samples by body site
- name: Hadoop
  role: Scalable MapReduce framework on which Libra runs; libra-all.jar is invoked
    via `hadoop jar`
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:6c5e2a72f31f26e4a66142b053d1c2eb33f53bbc7a1b08918c11d437ea633d87
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# distance-metric-resolution-benchmarking

## Summary

Benchmark how alternative metagenome distance metrics (Mash MinHash, Libra Cosine Similarity on k-mer abundance, Simka Jaccard-ab and Bray-Curtis) resolve biological structure — here, clustering of 48 HMP whole-genome shotgun read samples by body site — by running each tool on identical inputs and comparing the resulting sample clusterings.

## When to use

Use when choosing among metagenome comparison tools/metrics and you need evidence on which metric preserves body-site or community-composition structure rather than collapsing resolution.

## When NOT to use

- When the goal is a single tool's output quality rather than a head-to-head metric comparison — use the tool's own validation workflow instead
- When samples lack a known grouping (e.g., body site) against which clustering resolution can be judged; benchmarking resolution requires a ground-truth label
- When the input is not raw reads (e.g., already-computed distance matrices or sketch files) — re-running Mash/Libra/Simka on derived objects would not reproduce the read-level abundance comparison

## Inputs

- 48 paired whole-genome shotgun (WGS) read files from the HMIWGS (Human Microbiome Project) dataset, covering body sites including fornix, skin, and stool
- Tool configurations: Mash sketching parameters; Libra MAP-mode run (m=MAP, s=COSINESIMILARITY, w=LOGARITHM); Simka run for Jaccard-ab and Bray-Curtis

## Outputs

- All-vs-all distance/score matrices: one MinHash matrix (Mash), one Libra cosine-similarity score matrix, and two Simka matrices (Jaccard-ab, Bray-Curtis)
- Per-tool body-site clustering assignments for the 48 samples and a cross-tool comparison of clustering resolution and misclustering patterns

## How to apply

Run all tools on the same input set — here 48 paired whole-genome shotgun read files from the Human Microbiome Project (HMP) — so differences in clustering are attributable to the metric, not the data. Compute a Mash all-vs-all MinHash distance matrix from sketches of each sample; run Libra in MAP mode (m=MAP) with the COSINESIMILARITY scoring algorithm (s, default) and LOGARITHM weighting (w, default) to produce its score matrix; run Simka to obtain both Jaccard-ab and Bray-Curtis matrices. Cluster the 48 samples by body site under each of the four distance matrices and compare cluster assignments across tools. The rationale, per the Libra paper, is that data reduction, read count normalization, and presence/absence distance metrics diminish resolution, whereas Libra uses all reads to calculate k-mer abundance; validation is that body-site clustering is enhanced for Mash and Libra (Mash with decreased resolution) while samples remain misclustered under both Simka metrics, with some skin and stool samples clustering with fornix samples.

## Related tools

- **Libra** (Computes all-vs-all k-mer abundance scores (COSINESIMILARITY by default) on raw reads, run in MAP mode with LOGARITHM weighting; serves as the abundance-preserving metric under test) — https://github.com/iychoi/libra
- **Mash** (Sketches each sample and computes the all-vs-all MinHash distance matrix; baseline data-reduction metric compared against Libra)
- **Simka** (Computes Jaccard-ab and Bray-Curtis distance matrices between read sets; the presence/absence-style metrics that fail to cluster samples by body site)
- **Hadoop** (Scalable MapReduce framework on which Libra runs; libra-all.jar is invoked via `hadoop jar`)

## Examples

```
hadoop jar libra-all.jar core -s COSINESIMILARITY -m MAP -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Mash and Libra both show enhanced clustering of the 48 HMP WGS samples by body site
- Mash shows decreased resolution compared to Libra on the same samples
- Simka fails to cluster samples by body site under both Jaccard-ab and Bray-Curtis, with some skin and stool samples clustering with fornix samples
- Each of the four matrices is square all-vs-all over the same 48 samples, enabling direct row-wise comparison of cluster assignments
- Libra's score matrix is produced from a MAP-mode run (m=MAP) with COSINESIMILARITY scoring (s, default) and LOGARITHM weighting (w, default), written to the -o output directory

## Limitations

- Presence/absence and data-reduction metrics (e.g., Simka's Jaccard, Mash's MinHash on unique k-mers) diminish resolution: Mash underperforms Libra, and Simka misclusters skin and stool samples with fornix samples
- Benchmark outcome is tied to the HMP WGS read dataset and body-site grouping; results may not transfer to other community structures
- Libra requires a Hadoop environment and a preprocessing/indexing step before scoring, adding infrastructure overhead relative to sketch-based tools

## Evidence

- [abstract] Mash/Libra/Simka resolution claim: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [abstract] Libra uses all reads for k-mer abundance: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset"
- [intro] HMP 48-sample WGS benchmark set: "48 samples (16S rRNA and whole-genome shotgun sequencing [WGS]) from the Human Microbiome Project (HMP)"
- [methods] Scoring options including COSINESIMILARITY default: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] Libra MAP-mode invocation with weighting and output dir: [evidence span withheld — anchor status: external_doc]
- [intro] Simka computes a distance matrix between metagenomes: "Recently, Simka [15] was developed to compute a distance matrix between metagenomes"
