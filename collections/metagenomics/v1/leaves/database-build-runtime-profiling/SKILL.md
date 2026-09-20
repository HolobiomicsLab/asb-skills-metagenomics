---
name: database-build-runtime-profiling
description: Use when benchmarking how much time and RAM metagenomic classifiers need
  to build indexing databases from growing reference genome collections (e.g., bacterial-only
  vs. all-kingdom food-seq databases), to select a tool that fits a fixed-memory workstation
  or cluster.
when_to_use_negative:
- The goal is classification accuracy (false-positive rate, quantification accuracy)
  rather than construction cost — use the accuracy benchmark workflow instead
- The input is already a built database needing query-phase speed/memory profiling
  rather than a fresh build from reference genomes
- The database fits comfortably in available RAM and construction cost is not a constraint
  — build-time profiling adds no decision value
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier whose build mode (with -o remove-overpopulated-features)
    is timed and memory-profiled; fastest/lowest-memory builder in the benchmark
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark variant built with 16 executors (database stored in HDFS); used
    to measure cluster-mode construction cost
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Comparison classifier; its database build exceeded the 512 GB workstation
    memory on AFS31RS90 and is reported as '-'
- name: Kraken2
  role: Comparison classifier; k-mer database build time and peak memory recorded
    (e.g., 5h 55m / 134 GB on AFS31RS90)
- name: Kraken2+Bracken
  role: Comparison classifier plus abundance-estimation layer; heaviest build (e.g.,
    17h 44m / 329 GB on AFS31RS90)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:c0421e826fdd94dbec849454b9aef44e42a8fc738c98cf9f99399dea4cb18615
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# database-build-runtime-profiling

## Summary

Profile reference-database construction wall-clock time and peak main memory for k-mer/minhash metagenomic classifiers (AFS-MetaCache/MetaCacheSpark vs. CLARK, Kraken2, Kraken2+Bracken) across the AFS20, AFS31, AFS20RS90 and AFS31RS90 genome collections, mirroring Table 7 of the all-food-sequencing benchmark.

## When to use

Use when benchmarking how much time and RAM metagenomic classifiers need to build indexing databases from growing reference genome collections (e.g., bacterial-only vs. all-kingdom food-seq databases), to select a tool that fits a fixed-memory workstation or cluster.

## When NOT to use

- The goal is classification accuracy (false-positive rate, quantification accuracy) rather than construction cost — use the accuracy benchmark workflow instead
- The input is already a built database needing query-phase speed/memory profiling rather than a fresh build from reference genomes
- The database fits comfortably in available RAM and construction cost is not a constraint — build-time profiling adds no decision value

## Inputs

- Reference genome collections for four databases: AFS20, AFS31, AFS20RS90, AFS31RS90 (bacterial + all-kingdom, with and without reduced/redundant sets)
- FASTA/FASTQ reference genome sequences with taxonomy mapping for AFS-MetaCache/MetaCacheSpark builds
- A large-RAM workstation (e.g., 512 GB) or a Spark cluster with YARN and HDFS for MetaCacheSpark
- Resource monitor (/usr/bin/time -v or equivalent) for peak memory and wall-clock capture

## Outputs

- Build wall-clock time (e.g., 3h 10m for AFS-MetaCache on AFS31RS90) per tool x database combination
- Peak build memory (e.g., 135 GB for AFS-MetaCache, '-' for CLARK exceeding 512 GB) per tool x database combination
- A Table 7-style TSV with 4 tools x 4 databases rows and 5 metric fields (build time, peak build memory, query time, query speed, query memory)

## How to apply

Build each of the four reference databases (AFS20, AFS31, AFS20RS90, AFS31RS90) with AFS-MetaCache in build mode, enabling the remove-overpopulated-features option (-o) so that overpopulated features are pruned and classification-phase errors are avoided, and capture wall-clock time and peak memory with a resource monitor such as /usr/bin/time -v. Repeat the builds with CLARK, Kraken2, and Kraken2+Bracken under their default construction settings, recording build time and peak memory per tool/database combination. For the Spark implementation, launch MetaCacheSpark via spark-submit with 16 executors, storing the database in HDFS. Record any runs that exceed the workstation memory (in the paper, CLARK exceeded 512 GB on AFS31RS90 and is marked '-') rather than guessing. Tabulate build time and peak build memory per tool x database row and compare relative trends (AFS-MetaCache/MetaCacheSpark fastest and lowest-memory construction, Kraken2+Bracken slowest/heaviest) against the published relationships.

## Related tools

- **AFS-MetaCache** (C++ workstation classifier whose build mode (with -o remove-overpopulated-features) is timed and memory-profiled; fastest/lowest-memory builder in the benchmark) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark variant built with 16 executors (database stored in HDFS); used to measure cluster-mode construction cost) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Comparison classifier; its database build exceeded the 512 GB workstation memory on AFS31RS90 and is reported as '-')
- **Kraken2** (Comparison classifier; k-mer database build time and peak memory recorded (e.g., 5h 55m / 134 GB on AFS31RS90))
- **Kraken2+Bracken** (Comparison classifier plus abundance-estimation layer; heaviest build (e.g., 17h 44m / 329 GB on AFS31RS90))

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o
```

## Evaluation signals

- Table exists with all 4 tools x 4 databases rows and all 5 metric fields populated, or an explicit '-' for runs exceeding workstation memory (as for CLARK on AFS31RS90)
- Relative trends match Table 7: AFS-MetaCache/MetaCacheSpark show the fastest and lowest-memory database construction; Kraken2+Bracken is the slowest/heaviest build
- Each build ran with a resource monitor attached, so peak memory is a measured value (not an estimate); wall-clock times reproducible on equivalent hardware
- AFS-MetaCache builds used the -o (remove-overpopulated-features) option so later classification runs cannot fail on overpopulated features
- AFS31RS90 (largest, all-kingdom with reduced sets) shows the largest build times/memories across all tools, consistent with database size ordering AFS20 < AFS20RS90 < AFS31 < AFS31RS90

## Limitations

- Tool implementations may require more RAM than the workstation provides (CLARK exceeded 512 GB on AFS31RS90), yielding missing measurements rather than numbers
- No software version numbers are given for Kraken2, CLARK, Bracken, or Spark, so exact reproduction of absolute runtimes across versions is not guaranteed
- MetaCacheSpark builds require a Hadoop ecosystem (YARN + HDFS) and Spark 2 / Java 8, so workstation measurements do not transfer directly to cluster measurements
- Absolute timings depend on the benchmark hardware; only relative trends are stated as robust by the paper

## Evidence

- [methods] Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [discussion] AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times: "AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times"
- [abstract] the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [readme] -o,--remove_overpopulated_features [BUILD] Uses remove overpopulated features when building.: "-o,--remove_overpopulated_features   [BUILD] Uses remove overpopulated features when building."
- [results] For the AFS31RS90 database, AFS-MetaCache built fastest (3h 10m, 135 GB peak); CLARK exceeded the 512 GB workstation memory: [evidence span withheld — anchor status: not_found]
