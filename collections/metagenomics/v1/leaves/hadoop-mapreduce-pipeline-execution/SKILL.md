---
name: hadoop-mapreduce-pipeline-execution
description: Use when you must run Libra's Hadoop indexing (preprocess) and scoring
  (core) stages on large FASTA/FASTQ metagenome collections to produce an all-vs-all
  cosine-similarity matrix, e.g., global-ocean virome comparisons.
when_to_use_negative:
- No Hadoop cluster is available and the sample set is small — a single-node Hadoop
  deployment for a handful of small samples adds setup overhead without benefit
- A sketch-based or presence/absence comparison is explicitly desired — note the article
  warns that data reduction and presence/absence metrics diminish resolution of large-scale
  comparative analyses
- Input is already a k-mer index or a distance matrix rather than raw/QC'd FASTA-FASTQ
  reads — the preprocess stage expects sequencing reads in /source_dir
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: 'Provides the MapReduce jobs actually executed: the preprocess (k-mer indexing)
    and core (scoring) stages run via `hadoop jar libra-all.jar`'
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce execution framework on which Libra's indexing and scoring
    stages are launched
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:cc21d5f5ebf258a037b7b33205e0122fc3a31acd599cefe75561b57edf0950bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hadoop-mapreduce-pipeline-execution

## Summary

Executes Libra's two-stage Hadoop MapReduce pipeline — k-mer indexing of QC'd FASTA/FASTQ reads followed by all-vs-all scoring — to compare massive metagenome collections (e.g., 43 Tara Ocean Viromes, ~4.2 billion reads) without data reduction, using every read for k-mer abundance.

## When to use

Use when you must run Libra's Hadoop indexing (preprocess) and scoring (core) stages on large FASTA/FASTQ metagenome collections to produce an all-vs-all cosine-similarity matrix, e.g., global-ocean virome comparisons.

## When NOT to use

- No Hadoop cluster is available and the sample set is small — a single-node Hadoop deployment for a handful of small samples adds setup overhead without benefit
- A sketch-based or presence/absence comparison is explicitly desired — note the article warns that data reduction and presence/absence metrics diminish resolution of large-scale comparative analyses
- Input is already a k-mer index or a distance matrix rather than raw/QC'd FASTA-FASTQ reads — the preprocess stage expects sequencing reads in /source_dir

## Inputs

- Quality-controlled sequencing reads in FASTA/FASTQ format (e.g., 43 Tara Ocean Viromes from the 2009-2011 Expedition, ~4.2 billion reads)
- A Hadoop cluster deployment (article benchmark: 10-node Hadoop cluster)
- Libra binary (libra-all.jar, pre-built release compiled with Java 7)

## Outputs

- K-mer index directory (/index_dir) with per-group index files
- All-vs-all pairwise similarity scores directory (/score_dir), e.g., 903 pairwise scores for 43 viromes
- Assembled cosine-similarity distance matrix and sample clustering (e.g., four temperature-gradient groupings of the 43 TOVs)

## How to apply

First run the preprocessing/indexing stage on QC'd reads (e.g., the Brum et al.-filtered 43 Tara Ocean Viromes): `hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir`, choosing k-mer size (-k 20), reducer count (-t), index group size (-s, 10 GB default), and k-mer filter (-f NOTUNIQUE default, or STDDEV/STDDEV2). Then run the scoring stage on the index: `hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir`, keeping the defaults COSINESIMILARITY scoring algorithm, MAP run mode, and LOGARITHM weighting (alternatives: BRAYCURTIS, JENSENSHANNON; BOOLEAN/NATURAL weights; REDUCE mode). The rationale is that Cosine Similarity computes distance from sequence composition and abundance while normalizing for sequencing depth, and Libra deliberately uses all reads rather than the sketching/reduction of Mash or Sourmash. Finally, assemble the pairwise scores from /score_dir into an all-vs-all distance matrix and cluster the samples. For the TOV benchmark, verify the run reproduces the four temperature-gradient clusters (Fig. 5) and the reported runtime profile: k-mer matching within ~1.5 h and end-to-end ~18 h on a 10-node Hadoop cluster (Table 1).

## Related tools

- **Libra** (Provides the MapReduce jobs actually executed: the preprocess (k-mer indexing) and core (scoring) stages run via `hadoop jar libra-all.jar`) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce execution framework on which Libra's indexing and scoring stages are launched)

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir
```

## Evaluation signals

- The scoring stage completed with defaults confirmed: COSINESIMILARITY algorithm, MAP run mode, LOGARITHM weighting written to /score_dir
- The score directory contains a score for every sample pair (all-vs-all completeness, e.g., 43 TOVs = 903 pairs)
- Pairwise cosine-similarity values fall in the valid [0, 1] range for non-negative k-mer abundance vectors
- Runtime profile matches Table 1: k-mer matching within ~1.5 hours and end-to-end comparison within ~18 hours on a 10-node Hadoop cluster
- Clustering the assembled distance matrix reproduces the four temperature-gradient groupings of the 43 Tara Ocean Viromes (Fig. 5)

## Limitations

- Index grouping is constrained by the -s (min group size, 10 GB default) and -g (max 20 groups) options; exceeding the group cap forces groups to be combined
- The default k-mer filter is NOTUNIQUE; STDDEV/STDDEV2 filtering choices change which k-mers enter the index
- The pre-built binary was compiled with Java 7, so cluster Java/Hadoop versions must be compatible
- The article notes that data reduction, read count normalization, and presence/absence distance metrics diminish resolution — Libra's full-read MapReduce approach trades compute (and Hadoop infrastructure) for resolution

## Evidence

- [abstract] Libra uses a scalable Hadoop framework for massive metagenome comparisons: "Libra uses a scalable Hadoop framework for massive metagenome comparisons"
- [abstract] Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth: "Cosine Similarity for calculating the distance using sequence composition and abundance while normalizing for sequencing depth"
- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir: "hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir"
- [readme] hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir: [evidence span withheld — anchor status: external_doc]
