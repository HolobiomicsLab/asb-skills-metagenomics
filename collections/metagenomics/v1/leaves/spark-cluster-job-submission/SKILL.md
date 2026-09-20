---
name: spark-cluster-job-submission
description: Use when a reference-database build or read-classification workload is
  too large for a workstation and must be scaled out to a YARN/HDFS Hadoop Spark cluster,
  e.g. MetaCacheSpark jobs with 8–64 executors.
when_to_use_negative:
- When a single workstation has enough RAM to build the database unpartitioned — the
  C++ AFS-MetaCache tool is faster and simpler than launching a Spark cluster job
- When you need MetaCacheSpark modes beyond build/query (add, info, annotate are listed
  but only build and query are supported)
- When the reference database is small enough that cluster orchestration overhead
  outweighs the memory/scalability benefit of partitioning
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaCacheSpark
  role: Spark application submitted via spark-submit to build partitioned reference
    databases in HDFS and classify reads across cluster executors
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Big Data engine on a YARN-managed cluster that distributes the build and query
    work over 8–64 executors
- name: AFS-MetaCache
  role: C++ workstation baseline whose partitioned build/query timings are compared
    against the Spark cluster submissions
  repo: https://muellan.github.io/metacache/afs.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:e5fa309bf2878c51a0f12560ce6f081a0c7360073054948ece623cd60eec3dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# spark-cluster-job-submission

## Summary

Submitting MetaCacheSpark build and query jobs to a YARN-managed Apache Spark cluster via spark-submit, with executor counts (8–64) and partitions tuned to trade peak memory against build/query speed for large reference databases such as AFS31RS90.

## When to use

Use when a reference-database build or read-classification workload is too large for a workstation and must be scaled out to a YARN/HDFS Hadoop Spark cluster, e.g. MetaCacheSpark jobs with 8–64 executors.

## When NOT to use

- When a single workstation has enough RAM to build the database unpartitioned — the C++ AFS-MetaCache tool is faster and simpler than launching a Spark cluster job
- When you need MetaCacheSpark modes beyond build/query (add, info, annotate are listed but only build and query are supported)
- When the reference database is small enough that cluster orchestration overhead outweighs the memory/scalability benefit of partitioning

## Inputs

- Reference genome collection in FASTA format (e.g. the AFS31RS90 all-food reference set)
- Read dataset to classify in FASTA/FASTQ (e.g. the KAL_D read set, paired or single reads)
- YARN-managed Apache Spark cluster with HDFS storage (Spark 2, Java 8)
- MetaCacheSpark-0.4.0.jar built with Maven

## Outputs

- MetaCache minhashing reference database stored in HDFS (one per executor/partition configuration)
- Classification results for the query read set written to HDFS
- Measured build time, peak memory, and query speed per configuration (for results_table.csv)

## How to apply

Clone and package MetaCacheSpark with Maven (`mvn package`) to obtain MetaCacheSpark-0.4.0.jar, then submit it with spark-submit on a YARN-managed Spark cluster backed by HDFS. Use `-m build` with the `-o` (remove-overpopulated-features) option and a chosen executor count (8, 16, 32, or 64 in the paper, e.g. 15G executor memory and 20G driver memory), storing the resulting database in HDFS; use `-p` to set the number of partitions and `-e` to repartition sequences among executors. For classification, run `-m query` with the same executor count plus `-n` threads per executor (e.g. 16 executors with 4 threads per executor) and write classification results to HDFS. Increase executors to reduce build time and peak per-executor memory, mirroring the paper's Table 8 scaling where 4-partition AFS-MetaCache cut peak memory from 135 GB to 52 GB at the cost of query speed dropping from 7.2 to 2.5 MR/m.

## Related tools

- **MetaCacheSpark** (Spark application submitted via spark-submit to build partitioned reference databases in HDFS and classify reads across cluster executors) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Big Data engine on a YARN-managed cluster that distributes the build and query work over 8–64 executors)
- **AFS-MetaCache** (C++ workstation baseline whose partitioned build/query timings are compared against the Spark cluster submissions) — https://muellan.github.io/metacache/afs.html

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -p 16
```

## Evaluation signals

- A successful build job leaves a MetaCache database stored in HDFS, as in the paper's workflow where 'the database will be stored in HDFS'
- Peak main memory per executor stays within the configured limit (e.g. 15G executor memory, 20G driver memory) and decreases with more executors, reproducing Table 8 scaling
- Query jobs run with the documented pairing of 16 executors and 4 threads per executor and produce classification output in HDFS
- Compiled build time, peak memory, and query speed for all 7 configurations (AFS-MetaCache 1/2/4 partitions; MetaCacheSpark 8/16/32/64 executors) show the paper's trade-off, e.g. 135 GB → 52 GB peak memory with query speed falling from 7.2 to 2.5 MR/m at 4 partitions
- The remove-overpopulated-features option (-o) is enabled in build mode as in the paper's benchmark protocol

## Limitations

- MetaCacheSpark only supports build and query modes; other metacache modes (add, info, annotate) are listed but not supported
- Hardware specifications of the big data cluster used for the paper's benchmarks are not provided, so absolute timings may not transfer
- Query speed drops substantially when partitioning reduces memory (7.2 MR/m → 2.5 MR/m at 4 partitions), so partitioning trades speed for scalability
- Requires a Hadoop ecosystem cluster with YARN, HDFS, Spark 2, Java 8, Maven 3, and a C++14 compiler for the native/JNI code

## Evidence

- [methods] Execute MetaCacheSpark for building with 16 executors. The database will be stored in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [methods] Execute MetaCacheSpark for classification with 16 executors and 4 threads per executor.: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [results] Partitioning the AFS31RS90 database into four partitions reduces peak main memory from 135 GB to 52 GB while query speed drops from 7.2 MR/m to 2.5 MR/m: [evidence span withheld — anchor status: not_found]
- [readme] Requirements include a Big Data cluster with YARN and HDFS (Hadoop ecosystem), Spark 2, Java 8, Maven 3, and a C++14 compiler: "* A Big Data cluster with YARN and HDFS ([Hadoop][2] ecosystem)."
- [readme] Examples of how to launch MetaCacheSpark for building and querying are available at the script directory.: "Examples of how to launch **MetaCacheSpark** for building and querying are available at the *script* directory."
