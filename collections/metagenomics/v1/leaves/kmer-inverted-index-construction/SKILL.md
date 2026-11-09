---
name: kmer-inverted-index-construction
description: Use when raw metagenomic FASTA/FASTQ reads must be converted into a k-mer-indexed
  representation (e.g., k=20) as the input index for Libra's all-vs-all sample distance
  computation at scale.
when_to_use_negative:
- Input is already a k-mer index from a prior Libra preprocess run (proceed directly
  to the scoring/HM stage)
- Data are short of read-level resolution needs where sketching tools like Mash/Sourmash
  suffice and full k-mer abundance indexing is unnecessary
- A non-cosine distance metric is required and built via different preprocessing assumptions
  (Libra scoring offers BRAYCURTIS or JENSENSHANNON as alternatives to the default
  COSINESIMILARITY)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Provides the preprocess command that builds the k-mer inverted index from
    FASTA/FASTQ input for subsequent all-vs-all scoring
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Scalable MapReduce framework on which the k-mer indexing job runs ('hadoop
    jar libra-all.jar preprocess ...')
  repo: https://github.com/iychoi/libra
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:3e48dd91e1582c71ac4de1513f0b9f7341f5cbb84958b3d3713114d3a02e5f41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# kmer-inverted-index-construction

## Summary

Build Libra's k-mer inverted index from FASTA/FASTQ metagenomes via a Hadoop preprocessing step, so all sample k-mer frequency vectors are indexed by k-mer for scalable all-vs-all cosine similarity scoring.

## When to use

Use when raw metagenomic FASTA/FASTQ reads must be converted into a k-mer-indexed representation (e.g., k=20) as the input index for Libra's all-vs-all sample distance computation at scale.

## When NOT to use

- Input is already a k-mer index from a prior Libra preprocess run (proceed directly to the scoring/HM stage)
- Data are short of read-level resolution needs where sketching tools like Mash/Sourmash suffice and full k-mer abundance indexing is unnecessary
- A non-cosine distance metric is required and built via different preprocessing assumptions (Libra scoring offers BRAYCURTIS or JENSENSHANNON as alternatives to the default COSINESIMILARITY)

## Inputs

- FASTA/FASTQ metagenomic sequencing read files (source directory)
- Parameters: k-mer size (k), task/reducer count (t), k-mer filter (f: NONE | STDDEV | STDDEV2 | NOTUNIQUE), group size (s), max groups (g)

## Outputs

- k-mer inverted index under /index_dir, storing each sample's k-mer frequency vector keyed by k-mer (one index file per file group)

## How to apply

Run Libra's preprocessing stage on Hadoop over a directory of FASTA/FASTQ source files: 'hadoop jar libra-all.jar preprocess' with k-mer size k (README example uses k=20), number of tasks/reducers t (1 by default, 8 in the example), and output directory /index_dir. Decide whether to apply a k-mer filter via the f option (NOTUNIQUE by default; alternatives NONE, STDDEV, STDDEV2), and control index file partitioning with the s option (min group size, 10GB default) and g option (max groups, 20 default), so each file group yields a separate index file. The rationale is that the downstream COSINESIMILARITY sweep-line scorer requires a single scan of an inverted index keyed by k-mer, and Libra deliberately uses all reads (no sketching or presence/absence reduction) to preserve resolution. The resulting /index_dir then feeds directly into the scoring command 'HM -o /score_dir /index_dir'.

## Related tools

- **Libra** (Provides the preprocess command that builds the k-mer inverted index from FASTA/FASTQ input for subsequent all-vs-all scoring) — https://github.com/iychoi/libra
- **Hadoop** (Scalable MapReduce framework on which the k-mer indexing job runs ('hadoop jar libra-all.jar preprocess ...')) — https://github.com/iychoi/libra

## Examples

```
hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir
```

## Evaluation signals

- The output directory contains index files, one per file group as governed by the s (min 10GB default) and g (max 20 default) grouping options
- Running the scoring stage (HM) on /index_dir with s=COSINESIMILARITY completes without errors and emits an all-vs-all score matrix into /score_dir
- Self-comparison invariant: a dataset compared against a duplicate of itself yields a distance below 0.0001
- Chosen k (e.g., 20) and k-mer filter (default NOTUNIQUE) were explicitly set or accepted, consistent with the intended resolution for the dataset

## Limitations

- Index grouping defaults assume large inputs (10GB min group size, up to 20 groups); small datasets need parameter adjustment so groups are not combined or left oversized
- The k-mer filter choice (NOTUNIQUE by default; STDDEV/STDDEV2/NONE alternatives) changes which k-mers are retained and thus affects downstream distances
- Very large indexes may require falling back from MAP to REDUCE run mode at the scoring stage

## Evidence

- [readme] k-mer filter options: "f : kmer filter algorithm. NONE | STDDEV (standard deviation) | STDDEV2 (two's standard deviation) | NOTUNIQUE (default)"
- [readme] preprocess invocation: "hadoop jar libra-all.jar preprocess -k 20 -t 8 -o /index_dir /source_dir"
- [abstract] rationale for using all reads in a scalable index: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [abstract] purpose of the index for all-vs-all comparison: "performs an all-vs-all comparison of metagenomes for precise clustering based on their k-mer content"
