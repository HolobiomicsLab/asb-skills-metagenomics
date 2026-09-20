---
name: ocean-virome-community-analysis
description: Use when the goal is all-vs-all comparison or clustering of many large
  metagenomes/viromes (billions of reads) by k-mer content at full read depth, e.g.
  reproducing Tara Ocean Virome temperature-structured clustering (Fig. 5).
when_to_use_negative:
- The dataset is small or the analysis tolerates sketch-based approximations — the
  article notes faster MinHash tools like Mash/Sourmash, but Libra's value is full-depth
  comparison, so it is the wrong choice if speed at reduced resolution is acceptable.
- Input is already a reduced feature table or presence/absence matrix — the article
  warns that 'presence/absence distance metrics greatly diminish the resolution of
  large-scale comparative analyses', so this k-mer-based full-read workflow should
  not be applied to pre-summarized data.
- The goal is taxonomic or functional profiling of individual viromes rather than
  all-vs-all similarity and clustering across samples.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Performs the all-vs-all metagenome comparison based on k-mer content via its
    preprocess (indexing) and core (scoring) Hadoop stages, producing the cosine-similarity
    distance matrix used for clustering.
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which Libra runs; a 10-node cluster must complete
    k-mer matching within ~1.5 h and the end-to-end TOV comparison within ~18 h (Table
    1).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:5e2d187ba3cd7064d25419f0e725b3c70a427f21ed2ef8ccc18b96b5a462f699
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ocean-virome-community-analysis

## Summary

Reproduce Libra's all-vs-all k-mer comparison of the 43 Tara Ocean Viromes (~4.2 billion reads, 2009–2011 Expedition) on a Hadoop cluster to cluster global ocean viral populations and recover the four temperature-gradient groupings reported in Fig. 5.

## When to use

Use when the goal is all-vs-all comparison or clustering of many large metagenomes/viromes (billions of reads) by k-mer content at full read depth, e.g. reproducing Tara Ocean Virome temperature-structured clustering (Fig. 5).

## When NOT to use

- The dataset is small or the analysis tolerates sketch-based approximations — the article notes faster MinHash tools like Mash/Sourmash, but Libra's value is full-depth comparison, so it is the wrong choice if speed at reduced resolution is acceptable.
- Input is already a reduced feature table or presence/absence matrix — the article warns that 'presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses', so this k-mer-based full-read workflow should not be applied to pre-summarized data.
- The goal is taxonomic or functional profiling of individual viromes rather than all-vs-all similarity and clustering across samples.

## Inputs

- QC-filtered metagenomic/viral read files (FASTA/FASTQ) for the 43 Tara Ocean Viromes from the ENA 2009-2011 Expedition (~4.2 billion reads)
- Libra binary (libra-all.jar, e.g. v1.2.8) run under Hadoop on a 10-node cluster

## Outputs

- k-mer index directory (/index_dir) produced by the preprocess stage
- All-vs-all cosine-similarity scores (/score_dir) assembled into a 43x43 distance matrix
- Clustering of the 43 viromes reproducing the four temperature-gradient groupings of Fig. 5

## How to apply

Download the 43 Tara Ocean Virome read sets from ENA and apply the Brum et al. quality-control filtering protocol. Build the k-mer index by running Libra's preprocess/indexing stage on the QC'd FASTA/FASTQ reads (e.g. k-mer size 20, default NOTUNIQUE k-mer filter, output to /index_dir), which creates a separate index file per file group (10 GB default group size, 20 groups max). Run the Libra scoring stage in MAP run mode (default) with the COSINESIMILARITY algorithm (default) and LOGARITHM weighting (default), writing to /score_dir; Cosine Similarity is chosen because it considers genetic distance and microbial abundance simultaneously while normalizing for sequencing depth, and Libra deliberately uses all reads rather than sketching, data reduction, or presence/absence metrics, which the authors show greatly diminish resolution. Assemble the all-vs-all cosine-similarity distance matrix from /score_dir and cluster the 43 viromes; success means the four temperature-gradient groupings of Fig. 5 emerge, with samples from different Longhurst provinces but the same temperature gradient clustering together.

## Related tools

- **Libra** (Performs the all-vs-all metagenome comparison based on k-mer content via its preprocess (indexing) and core (scoring) Hadoop stages, producing the cosine-similarity distance matrix used for clustering.) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which Libra runs; a 10-node cluster must complete k-mer matching within ~1.5 h and the end-to-end TOV comparison within ~18 h (Table 1).)

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir && hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Clustering of the 43 viromes recovers the four temperature-gradient groupings of Fig. 5, with samples from different Longhurst provinces but the same temperature gradient clustering together.
- K-mer matching completes within ~1.5 hours and the end-to-end comparison within ~18 hours on a 10-node Hadoop cluster, matching Table 1.
- The preprocess stage emits an index directory whose per-group index files respect the '-s 10GB' group-size and '-g 20' max-group defaults.
- The scoring output in /score_dir corresponds to the default COSINESIMILARITY algorithm with LOGARITHM weighting in MAP run mode, and the assembled matrix is symmetric all-vs-all over the 43 samples.

## Limitations

- Compute cost scales with full read depth: the TOV run requires a 10-node Hadoop cluster and ~18 hours end-to-end, so smaller setups may be impractical.
- Libra's full-depth design trades speed for resolution relative to sketch-based tools (Mash, Sourmash) and reduced methods (Compareads, Commet, Simka).
- The preprocess '-g' option combines groups when the number of 10 GB index groups would exceed the 20-group default, which constrains index partitioning for very large inputs.

## Evidence

- [abstract] We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content.: "We developed a tool called Libra that performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content."
- [abstract] methods that diminish resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [intro] Cosine Similarity rationale: "We demonstrate that Cosine Similarity, which is widely used in document clustering and information retrieval, is a good distance metric for comparing datasets to consider genetic distance and"
- [readme] README scoring command: [evidence span withheld — anchor status: external_doc]
- [readme] README preprocessing command: "hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir"
