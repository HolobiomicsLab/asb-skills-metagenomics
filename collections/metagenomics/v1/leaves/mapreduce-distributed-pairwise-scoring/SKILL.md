---
name: mapreduce-distributed-pairwise-scoring
description: Use when you need an all-vs-all distance matrix between many (FASTA/FASTQ)
  metagenomic samples at scale, and a per-pair loop is infeasible; a k-mer index directory
  already exists or can be built.
when_to_use_negative:
- Input data has already been reduced, normalized by read count, or binarized to presence/absence
  — the paper shows such data reduction and presence/absence distance metrics greatly
  diminish the resolution of large-scale comparative analyses
- You only need a small number of pairwise comparisons (n << all-vs-all); a direct
  pairwise scan is simpler than a MapReduce sweep-line over an inverted index
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Runs the all-vs-all pairwise scoring (HM/core command) with COSINESIMILARITY
    over the k-mer index to produce the distance matrix
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Distributed MapReduce framework that executes the sweep-line scoring in MAP
    (default) or REDUCE mode for massive datasets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:182902070f3b5b689f4f80556818758854c60d38315f625ce11bc2e9e5ffc2b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mapreduce-distributed-pairwise-scoring

## Summary

Implements Libra's Hadoop MapReduce scoring stage, which computes the all-vs-all pairwise distance matrix between metagenomic samples using a sweep-line over a k-mer inverted index with the COSINESIMILARITY scoring algorithm, avoiding quadratic scans of individual vector pairs.

## When to use

Use when you need an all-vs-all distance matrix between many (FASTA/FASTQ) metagenomic samples at scale, and a per-pair loop is infeasible; a k-mer index directory already exists or can be built.

## When NOT to use

- Input data has already been reduced, normalized by read count, or binarized to presence/absence — the paper shows such data reduction and presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses
- You only need a small number of pairwise comparisons (n << all-vs-all); a direct pairwise scan is simpler than a MapReduce sweep-line over an inverted index

## Inputs

- Libra k-mer inverted index directory (/index_dir) built from FASTA/FASTQ sample files by the preprocess stage
- Scoring options: s (COSINESIMILARITY | BRAYCURTIS | JENSENSHANNON), m (MAP | REDUCE), w (LOGARITHM | BOOLEAN | NATURAL), -o output directory

## Outputs

- All-vs-all sample distance matrix (e.g. cosine distance between k-mer frequency vectors) written into /score_dir

## How to apply

First build (or reuse) a k-mer inverted index keyed by k-mer that stores each sample's k-mer frequency vector, e.g. via Libra's preprocess stage into /index_dir. Then run Libra's scoring command 'HM -o /score_dir /index_dir' (hadoop jar libra-all.jar core ...) with s=COSINESIMILARITY (the default scoring algorithm) to sweep k-mers in MAP mode (m=MAP, default), accumulating the dot-product term D and per-sample magnitude term M for every sample pair in a single pass instead of scanning each vector pair separately. Choose the weighting via -w: LOGARITHM (default, 1+log f), NATURAL (raw k-mer frequency f), or BOOLEAN presence/absence — favor LOGARITHM or NATURAL since the paper shows presence/absence metrics greatly diminish resolution. Use REDUCE mode (m=REDUCE) as a fallback for very large indexes where map-side scoring does not fit. The output is the all-vs-all distance matrix written into /score_dir.

## Related tools

- **Libra** (Runs the all-vs-all pairwise scoring (HM/core command) with COSINESIMILARITY over the k-mer index to produce the distance matrix) — https://github.com/iychoi/libra
- **Hadoop** (Distributed MapReduce framework that executes the sweep-line scoring in MAP (default) or REDUCE mode for massive datasets)

## Examples

```
hadoop jar libra-all.jar core -s COSINESIMILARITY -m MAP -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Comparing a dataset against a duplicate of itself yields a distance below 0.0001
- Output /score_dir contains a complete all-vs-all matrix with one distance for every sample pair
- The scoring algorithm reported/used is COSINESIMILARITY (s=COSINESIMILARITY, the default)
- Runtime scales roughly linearly with a single sweep over the index rather than quadratically with sample count (the sweep accumulates D and M for all pairs in one scan)
- Weighting scheme matches the requested option (w=LOGARITHM/NATURAL/BOOLEAN) rather than silently falling back to BOOLEAN

## Limitations

- Very large indexes may require falling back from MAP mode to REDUCE mode (m=REDUCE) if map-side scoring does not scale
- Alternatives COSINESIMILARITY vs BRAYCURTIS vs JENSENSHANNON are available; correctness of the distance interpretation only holds for the chosen metric and weighting
- Quality depends on the upstream preprocess parameters (k-mer size, kmer filter NONE|STDDEV|STDDEV2|NOTUNIQUE, group size/max groups) used to build /index_dir

## Evidence

- [abstract] Libra uses a scalable Hadoop framework for massive metagenome comparisons: "Libra uses a scalable Hadoop framework for massive metagenome comparisons"
- [abstract] all-vs-all comparison of metagenomes based on k-mer content: "performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content"
- [readme] scoring algorithm options: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] scoring invocation and options: [evidence span withheld — anchor status: external_doc]
