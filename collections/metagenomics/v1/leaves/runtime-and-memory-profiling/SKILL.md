---
name: runtime-and-memory-profiling
description: Use when benchmarking a large reference database build or classification
  run — e.g. comparing partitioned vs. unpartitioned builds of the AFS31RS90 database
  across AFS-MetaCache partitions or MetaCacheSpark executor counts.
when_to_use_negative:
- The database fits comfortably in workstation RAM — README notes partitioning is
  for cases where 'you don't have enough RAM' (default settings need ≥128GB for full
  NCBI RefSeq)
- The goal is classification accuracy (false-positive rate, quantification accuracy
  vs. Kraken2/CLARK/Bracken) rather than runtime and memory scaling
- The database is small or the analysis uses a different tool (e.g. Kraken2, CLARK)
  whose memory profile this profiling workflow does not measure
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier used to build 1-, 2-, and 4-partition AFS31RS90
    databases and measure build time, peak memory, and query speed
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark-based classifier run with 8-64 executors to profile distributed build
    and query performance on YARN with databases stored in HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Execution engine (via spark-submit on YARN) whose executor count and memory
    settings (e.g. 15G executor, 20G driver) are varied in the profiling benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:f1579302f62ae110fe4c03bc210821cee3b8c9aa7c0367454cc31b32604bc1f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# runtime-and-memory-profiling

## Summary

Profiles build time, peak main memory consumption, and query speed (million reads per minute) of a metagenomic classifier across database configurations. In the AFS-MetaCache/MetaCacheSpark paper this quantifies how partitioning the AFS31RS90 reference database trades memory (135 GB → 52 GB with 4 partitions) for slower builds and queries (Table 8).

## When to use

Use when benchmarking a large reference database build or classification run — e.g. comparing partitioned vs. unpartitioned builds of the AFS31RS90 database across AFS-MetaCache partitions or MetaCacheSpark executor counts.

## When NOT to use

- The database fits comfortably in workstation RAM — README notes partitioning is for cases where 'you don't have enough RAM' (default settings need ≥128GB for full NCBI RefSeq)
- The goal is classification accuracy (false-positive rate, quantification accuracy vs. Kraken2/CLARK/Bracken) rather than runtime and memory scaling
- The database is small or the analysis uses a different tool (e.g. Kraken2, CLARK) whose memory profile this profiling workflow does not measure

## Inputs

- AFS31RS90 reference genome collection for database construction
- KAL_D read dataset (classification/query input)
- MetaCacheSpark jar (e.g. MetaCacheSpark-0.4.0.jar) and a YARN-managed Spark cluster with HDFS
- AFS-MetaCache C++ workstation binary compiled with a C++14 compiler

## Outputs

- results_table.csv with build time, peak main memory, and query speed for 7 configurations (AFS-MetaCache 1/2/4 partitions; MetaCacheSpark 8/16/32/64 executors)
- Partitioned reference databases stored in HDFS (MetaCacheSpark) or on the workstation (AFS-MetaCache)
- Classification results written to HDFS for the KAL_D read set

## How to apply

Build the AFS31RS90 reference database with AFS-MetaCache using 1, 2, and 4 database partitions and record wall-clock build time and peak main memory per run. In parallel, build the same database with MetaCacheSpark via spark-submit in build mode (-m build) on a YARN-managed Spark cluster with 8, 16, 32, and 64 executors (paper example: 15G executor memory, 20G driver memory), enabling the remove-overpopulated-features option (-o), storing each database in HDFS. Classify the KAL_D read set in query mode (-m query) against each of the 7 resulting databases (MetaCacheSpark example: 16 partitions with 4 threads per executor). Compile build time, peak memory, and query speed in million reads per minute into results_table.csv and compare the scaling behavior against the paper's Table 8, where 4 partitions reduce peak memory from 135 GB to 52 GB while build time grows from 3h 10m to 3h 45m and query speed falls from 7.2 to 2.5 MR/m.

## Related tools

- **AFS-MetaCache** (C++ workstation classifier used to build 1-, 2-, and 4-partition AFS31RS90 databases and measure build time, peak memory, and query speed) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark-based classifier run with 8-64 executors to profile distributed build and query performance on YARN with databases stored in HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Execution engine (via spark-submit on YARN) whose executor count and memory settings (e.g. 15G executor, 20G driver) are varied in the profiling benchmark)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -p 16
```

## Evaluation signals

- Peak memory of AFS-MetaCache builds decreases with partition count: 135 GB (1 partition) → 52 GB (4 partitions)
- Build time increases only slightly with partitioning: 3h 10m → 3h 45m from 1 to 4 partitions
- Query speed on KAL_D drops with partitioning: 7.2 MR/m (1 partition) → 2.5 MR/m (4 partitions)
- All 7 configurations (AFS-MetaCache 1/2/4 partitions; MetaCacheSpark 8/16/32/64 executors) report build time, peak memory, and query speed, matching the scaling trend of Table 8
- Each MetaCacheSpark database build completes with output stored in HDFS and the -o (remove-overpopulated-features) option enabled

## Limitations

- Partitioning trades memory for speed: query speed falls from 7.2 to 2.5 MR/m when using 4 partitions instead of 1
- Hardware specifications of the big data cluster used for benchmarks are not provided in the article
- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark)
- Databases can only be queried with the same MetaCache variant and compilation settings used to build them (data type width/endianness differences can break cross-platform use)

## Evidence

- [abstract] the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [methods] Execute MetaCacheSpark for building with 16 executors. The database will be stored in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [discussion] AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times: "AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times"
- [readme] If you don't have enough RAM, you can use database partitioning.: [evidence span withheld — anchor status: external_doc]
- [readme] Uses remove overpopulated features when building.: "Uses remove overpopulated
                                          features when building."
