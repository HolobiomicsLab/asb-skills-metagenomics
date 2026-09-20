---
name: hdfs-distributed-sequence-partitioning
description: Use when reference genome collections are too large for a single-machine
  k-mer/minhash database build and input FASTA sequences plus taxonomy already reside
  in HDFS on a Hadoop cluster with Spark 2.
when_to_use_negative:
- Reference genome collection is small enough for the C++ workstation version (AFS-MetaCache
  / metacache), which builds databases with reduced memory without a cluster
- Sequences and taxonomy are not in HDFS or the cluster lacks YARN/HDFS (Hadoop ecosystem)
  — MetaCacheSpark requires a Big Data cluster with YARN and HDFS
- The goal is classification/querying rather than database construction — use MetaCacheSpark's
  -m query mode instead
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaCacheSpark
  role: Spark-based implementation that distributes reference sequences among executors,
    builds executor-local minhash C++ hash tables, merges them via map-reduce, and
    writes the partitioned database to HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Big Data engine (Spark 2 on YARN) that executes the build job, partitions
    sequences with -p and optionally -e repartition, and orchestrates the executors
- name: AFS-MetaCache
  role: C++ workstation alternative for building the same minhash database on a single
    machine when cluster-scale partitioning is unnecessary
  repo: https://muellan.github.io/metacache/afs.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:9411cd2faa87af71ae5e2eb4f0fb2a6e34cbf2ffcb0e43bb986848efd81a96d8
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
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hdfs-distributed-sequence-partitioning

## Summary

Build a partitioned metagenomic reference database on a Spark/YARN cluster by loading FASTA reference genomes from HDFS, distributing sequences across Spark executors (by Spark repartition or by sequence length), and sketching them into executor-local C++ hash tables via minhashing, with a global map-reduce merge and removal of overpopulated features.

## When to use

Use when reference genome collections are too large for a single-machine k-mer/minhash database build and input FASTA sequences plus taxonomy already reside in HDFS on a Hadoop cluster with Spark 2.

## When NOT to use

- Reference genome collection is small enough for the C++ workstation version (AFS-MetaCache / metacache), which builds databases with reduced memory without a cluster
- Sequences and taxonomy are not in HDFS or the cluster lacks YARN/HDFS (Hadoop ecosystem) — MetaCacheSpark requires a Big Data cluster with YARN and HDFS
- The goal is classification/querying rather than database construction — use MetaCacheSpark's -m query mode instead

## Inputs

- Reference genome sequence files (FASTA/FASTQ) stored in HDFS
- Taxonomy files stored in HDFS (path passed via -t)
- MetaCacheSpark-0.4.0.jar built with Maven from the cloned repository

## Outputs

- Partitioned MetaCacheSpark database of executor-local C++ hash tables written to HDFS (e.g. Database_16 at the user home)
- Database loadable in query mode (spark-submit with -m query) for downstream read classification

## How to apply

Clone the MetaCacheSpark repository and run 'mvn package' to produce MetaCacheSpark-0.4.0.jar (requires Java 8, Maven 3, Spark 2, and a C++14 compiler for the native/JNI metacache code). Upload reference genome sequences and taxonomy files to HDFS, then launch the build job via spark-submit on YARN with --class com.github.jmabuin.metacachespark.MetaCacheSpark, e.g. --num-executors 16 --executor-memory 15G --driver-memory 20G, passing -m build, -p 16 partitions, -o to remove overpopulated features (recommended when handling very large data to avoid classification-phase errors), and -t pointing to the taxonomy path in HDFS; use -e for Spark repartition-based sequence distribution, otherwise sequences are distributed according to their length. Each executor minhash-sketches its local sequences into executor-local C++ hash tables; a map-reduce global key-count merge then ensures each reference sequence's items reside in exactly one hash table, and features above the overpopulation deletion threshold are removed. Each executor-local hash table is written to HDFS so the partitioned database (e.g. Database_16) is stored at the user home.

## Related tools

- **MetaCacheSpark** (Spark-based implementation that distributes reference sequences among executors, builds executor-local minhash C++ hash tables, merges them via map-reduce, and writes the partitioned database to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Big Data engine (Spark 2 on YARN) that executes the build job, partitions sequences with -p and optionally -e repartition, and orchestrates the executors)
- **AFS-MetaCache** (C++ workstation alternative for building the same minhash database on a single machine when cluster-scale partitioning is unnecessary) — https://muellan.github.io/metacache/afs.html

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark --conf spark.network.timeout=10000000 --num-executors 16 --master yarn --executor-memory 15G --driver-memory 20G MetaCacheSpark-0.4.0.jar -m build -o -p 16 -t /path/to/taxonomy/in/hdfs/ Database_16 /path/to/input/sequences/in/hdfs/
```

## Evaluation signals

- The database (e.g. Database_16) exists in HDFS at the user home after the build job completes
- The database loads successfully in query mode (spark-submit with -m query against Database_16)
- All 16 partitions/hash tables were produced, matching the -p 16 setting, with each reference sequence's items residing in exactly one hash table
- Overpopulated features above the deletion threshold were removed because the -o (remove-overpopulated-features) flag was set
- Build was launched with the expected executor resources (--num-executors 16, --executor-memory 15G, --driver-memory 20G) per the README example

## Limitations

- MetaCacheSpark requires a Big Data cluster with YARN and HDFS plus Spark 2, Java 8, Maven 3, and a C++14 compiler; it is not a single-node tool
- Without the -o (remove-overpopulated-features) option on very large collections, classification-phase errors can occur
- If -e (Spark repartition) is omitted, sequences are distributed by length rather than via Spark repartitioning, which changes the partitioning behavior
- MetaCacheSpark only supports build and query modes (plus config-file options); other metacache modes are unavailable

## Evidence

- [abstract] Load reference genomes and build partitioned minhash database on Spark executors: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [readme] Build-mode sequence distribution across executors is controlled by -e: "-e: Indicates if the input reads are distributed among the cluster by using a Spark *repartition* or not. If not, sequences are distributed according their length."
- [readme] Remove-overpopulated-features option is recommended at build time for large data: "As theoretically, in the case of using **MetaCacheSpark** the user is dealing with a lot of data, this option should be used in order to avoid errors during the classification phase."
- [methods] Build example with 16 executors storing Database_16 in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [methods] Building MetaCacheSpark requires cloning and mvn package: "git clone https://github.com/jmabuin/MetaCacheSpark.git"
