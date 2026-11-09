---
name: database-partitioning-scalability-benchmarking
description: Use when a reference genome database (e.g. AFS31RS90, ~135 GB in RAM)
  exceeds single-node or single-executor memory and you need to quantify the time/memory/speed
  trade-off of splitting it into partitions or distributing it over Spark executors.
when_to_use_negative:
- The database already fits comfortably in workstation memory (e.g. ≤128 GB for the
  default RefSeq build), since partitioning only costs query speed without necessity.
- The task is tool accuracy benchmarking against CLARK, Kraken2, or Kraken2+Bracken
  rather than memory/time scaling of partitioned database construction.
- Query speed, not peak memory, is the binding constraint — partitioning degraded
  throughput from 7.2 to 2.5 MR/m in the paper's measurements.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation used to build the AFS31RS90 database with 1,
    2, and 4 partitions and measure single-node peak memory scaling
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark implementation used to build the same database on a YARN cluster with
    8–64 executors (-m build, -o) and query KAL_D reads into HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Distributed execution engine (spark-submit on YARN with HDFS storage) for
    the executor-scaling experiments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:f7907c4fd9cce1cee64a9ceb686ebaf667fd35f766cae037aa09b5c76164d550
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# database-partitioning-scalability-benchmarking

## Summary

Benchmark how partitioning a large minhashing reference database (AFS31RS90) affects build time, peak memory, and query speed for the C++ workstation tool AFS-MetaCache and the Spark-based MetaCacheSpark, reproducing the paper's Table 8 scaling behaviour.

## When to use

Use when a reference genome database (e.g. AFS31RS90, ~135 GB in RAM) exceeds single-node or single-executor memory and you need to quantify the time/memory/speed trade-off of splitting it into partitions or distributing it over Spark executors.

## When NOT to use

- The database already fits comfortably in workstation memory (e.g. ≤128 GB for the default RefSeq build), since partitioning only costs query speed without necessity.
- The task is tool accuracy benchmarking against CLARK, Kraken2, or Kraken2+Bracken rather than memory/time scaling of partitioned database construction.
- Query speed, not peak memory, is the binding constraint — partitioning degraded throughput from 7.2 to 2.5 MR/m in the paper's measurements.

## Inputs

- AFS31RS90 reference genome collection for database construction
- KAL_D read dataset (FASTA/FASTQ) for classification queries
- YARN-managed Apache Spark cluster with HDFS (Spark 2, Java 8) and MetaCacheSpark-0.4.0.jar
- C++ workstation build of AFS-MetaCache (metacache)

## Outputs

- AFS-MetaCache partitioned databases (1, 2, 4 partitions) with measured build times and peak main memory per configuration
- MetaCacheSpark databases stored in HDFS (8/16/32/64 executors) with build times and per-executor peak memory
- Classification results for KAL_D written to HDFS and per-configuration query speeds (MR/m)
- results_table.csv compiling build time, peak memory, and query speed for all 7 configurations reproducing Table 8

## How to apply

Build the AFS31RS90 reference database with AFS-MetaCache on a workstation three times using 1, 2, and 4 database partitions, recording build time and peak main memory per run. In parallel, build the same database with MetaCacheSpark via spark-submit in build mode (-m build) on a YARN-managed Spark cluster with 8, 16, 32, and 64 executors (e.g. 15G executor memory, 20G driver memory), enabling the remove-overpopulated-features option (-o) and storing each database in HDFS. Classify the KAL_D read set in query mode (-m query) against all seven database configurations, e.g. 16 partitions with 4 threads per executor for Spark. Rationale: partitioning trades query throughput for feasibility — the paper shows 4 partitions cut peak memory from 135 GB to 52 GB while build time rose only from 3h 10m to 3h 45m, but query speed dropped from 7.2 to 2.5 MR/m.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation used to build the AFS31RS90 database with 1, 2, and 4 partitions and measure single-node peak memory scaling) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark implementation used to build the same database on a YARN cluster with 8–64 executors (-m build, -o) and query KAL_D reads into HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Distributed execution engine (spark-submit on YARN with HDFS storage) for the executor-scaling experiments)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -p 16
```

## Evaluation signals

- AFS-MetaCache 1-partition run reproduces the paper's reference point: ~3h 10m build time, 135 GB peak memory, 7.2 MR/m query speed on KAL_D.
- 4-partition AFS-MetaCache run shows peak memory reduced to ~52 GB with build time rising only modestly to ~3h 45m and query speed dropping to ~2.5 MR/m.
- MetaCacheSpark build times and peak memory decrease monotonically (or near-monotonically) as executor count grows from 8 to 64; each database is stored successfully in HDFS.
- All 7 configurations (3 AFS-MetaCache partitions + 4 executor counts) yield complete rows in results_table.csv with build time, peak memory, and query speed matching Table 8's scaling behaviour.
- Query-mode runs complete for every configuration with classification results written to HDFS using the same KAL_D read set, ensuring speed values are directly comparable.

## Limitations

- Query throughput degrades substantially with more partitions: 7.2 MR/m at 1 partition vs 2.5 MR/m at 4 partitions for AFS-MetaCache.
- Hardware specifications of the big data cluster used for benchmarks are not provided in the article, limiting exact reproduction of MetaCacheSpark timings.
- Databases built with one variant of MetaCache (data-type sizes, compiler) may not be queryable by another build due to endianness and data-type width differences, per the metacache README.
- No software version numbers are given for benchmarked tools (Spark, Kraken2, CLARK, Bracken), complicating exact re-runs.

## Evidence

- [results] Partitioning reduces peak memory while modestly increasing build time and reducing query speed: [evidence span withheld — anchor status: not_found]
- [abstract] Partitioning enables reduced-memory processing of massive genome collections: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [methods] MetaCacheSpark build mode with executors storing database in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [methods] remove-overpopulated-features build option: "* **-o**: Uses the *remove-overpopulated-features*."
- [methods] Query-mode invocation with threads per executor: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
