---
name: hdfs-data-management
description: Use when running MetaCacheSpark build or query jobs on a Hadoop/Spark
  cluster and the reference database or classification results must be shared across
  Spark executors via HDFS rather than a workstation-local filesystem.
when_to_use_negative:
- Running the C++ AFS-MetaCache workstation tool, which builds and queries local,
  non-HDFS databases (1-4 partitions in main memory)
- Single-node analyses of small reference databases that fit in workstation RAM and
  do not need a Spark/Hadoop cluster
- Tools like Kraken2, CLARK, or Kraken2+Bracken that manage their own index storage
  formats
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaCacheSpark
  role: Spark-based metagenomic classifier whose build and query modes write databases
    and classification results to HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Execution engine launched via spark-submit on a YARN-managed cluster, distributing
    build/query work over 8-64 executors against HDFS-stored data
- name: AFS-MetaCache
  role: C++ workstation alternative that does NOT use HDFS; contrast tool for the
    partitioned-database benchmarks (Table 8)
  repo: https://muellan.github.io/metacache/afs.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:a95919549822a9c806b967fe6b595edfdd2fe38181c915249b385dfdd86faaa3
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hdfs-data-management

## Summary

Manages storage of metagenomic reference databases and classification outputs on the Hadoop Distributed File System (HDFS) when running MetaCacheSpark on a YARN-managed Spark cluster, so that build/query jobs with many executors can read and write the AFS31RS90-style reference databases in parallel.

## When to use

Use when running MetaCacheSpark build or query jobs on a Hadoop/Spark cluster and the reference database or classification results must be shared across Spark executors via HDFS rather than a workstation-local filesystem.

## When NOT to use

- Running the C++ AFS-MetaCache workstation tool, which builds and queries local, non-HDFS databases (1-4 partitions in main memory)
- Single-node analyses of small reference databases that fit in workstation RAM and do not need a Spark/Hadoop cluster
- Tools like Kraken2, CLARK, or Kraken2+Bracken that manage their own index storage formats

## Inputs

- Reference genome sequences (e.g. the AFS31RS90 all-food-sequencing genome collection) consumed in build mode
- Read datasets in FASTA/FASTQ form (e.g. the KAL_D read set) consumed in query mode
- config.properties configuration file (passable via the -c option)
- HDFS and YARN services on the cluster

## Outputs

- Partitioned MetaCache database stored in HDFS after build mode
- Per-read taxonomic classification results (with optional abundance estimation) written to HDFS after query mode

## How to apply

Provision a Big Data cluster with YARN and HDFS, since MetaCacheSpark depends on the Hadoop ecosystem. Run MetaCacheSpark through spark-submit with the MetaCacheSpark-0.4.0.jar produced by 'mvn package'; in build mode (-m build), with the remove-overpopulated-features option (-o) and a chosen partition count (-p), the constructed database is automatically stored in HDFS (e.g. 16 executors as in the paper's example). In query mode (-m query), classification results against that HDFS-stored database (e.g. KAL_D reads, 16 executors with 4 threads per executor) are likewise written to HDFS. The rationale is that partitioned databases large enough to need 8-64 executors (and reduced per-node memory, e.g. 15G executor / 20G driver memory) cannot live on a single workstation disk or RAM, so HDFS provides the shared, replicated store that each executor's partitions read from and write to.

## Related tools

- **MetaCacheSpark** (Spark-based metagenomic classifier whose build and query modes write databases and classification results to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Execution engine launched via spark-submit on a YARN-managed cluster, distributing build/query work over 8-64 executors against HDFS-stored data)
- **AFS-MetaCache** (C++ workstation alternative that does NOT use HDFS; contrast tool for the partitioned-database benchmarks (Table 8)) — https://muellan.github.io/metacache/afs.html

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -p 16 -e
```

## Evaluation signals

- After a build run, the partitioned database (e.g. -p 16) is present in HDFS and can be listed/read back by all executors
- Query mode completes against the HDFS-stored database and classification results appear in HDFS
- Peak memory per node stays within the allocated executor memory (e.g. 15G executor / 20G driver) rather than spilling beyond YARN containers
- Scaling behaviour matches Table 8: more executors or partitions reduce peak memory per node (e.g. 135 GB -> 52 GB for AFS-MetaCache 1 -> 4 partitions) at a cost in query speed (7.2 -> 2.5 MR/m)
- spark-submit completes with the correct jar (MetaCacheSpark-0.4.0.jar) and mode flag (-m build or -m query) without YARN container failures

## Limitations

- MetaCacheSpark only supports build and query modes; other metacache modes (add, info, annotate) are listed but the README notes limited support
- Hardware specifications of the big data cluster used for the paper's benchmarks are not provided, so absolute times may not be reproducible
- Databases can only be queried by a MetaCache variant compiled with the same data type settings (target ID, window ID, k-mer type widths)
- In rare cases databases built on one platform may fail on another due to endianness and data type width differences

## Evidence

- [methods] Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [readme] A Big Data cluster with YARN and HDFS ([Hadoop][2] ecosystem).: "A Big Data cluster with YARN and HDFS ([Hadoop][2] ecosystem)."
- [abstract] the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [methods] Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor.: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
