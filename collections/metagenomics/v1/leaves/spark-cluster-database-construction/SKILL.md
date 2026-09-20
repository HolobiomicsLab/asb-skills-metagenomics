---
name: spark-cluster-database-construction
description: Use when reference genome collections are too large for single-workstation
  k-mer database construction and a YARN/HDFS cluster with Spark 2 is available, requiring
  a partitioned minhash database instead of one monolithic file.
when_to_use_negative:
- Reference genome collection is small enough for a single-workstation database —
  use the C++ AFS-MetaCache or muellan/metacache CPU build instead of the Spark pipeline
- No YARN/HDFS big-data cluster is available (MetaCacheSpark requires Hadoop ecosystem,
  Spark 2, and HDFS for both input and output)
- Goal is classification of reads rather than database construction — run in query
  mode (-m query) against an already built database
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaCacheSpark
  role: 'Primary tool: Spark wrapper around metacache''s minhashing algorithm; distributes
    sequences to executors, builds executor-local C++ hash tables via JNI, merges
    and writes the partitioned database to HDFS'
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: 'Execution engine on YARN: provides executor distribution of sequences, map-reduce
    global key-count merge, and HDFS I/O'
- name: AFS-MetaCache
  role: C++ single-workstation counterpart producing equivalent minhash databases
    with reduced memory requirements; alternative when no cluster is needed
  repo: https://muellan.github.io/metacache/afs.html
- name: metacache
  role: Origin of the minhashing algorithm and native C++ hash-table code linked into
    MetaCacheSpark via JNI
  repo: https://github.com/muellan/metacache
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:d0d5940131191179ebd0156a03ef0d1d2eb3764df3a3d12ca68266c734463383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  - metacache
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# spark-cluster-database-construction

## Summary

Builds a partitioned metagenomic reference database on an Apache Spark/Hadoop cluster with MetaCacheSpark: reference genomes are read from HDFS, distributed across Spark executors, locally minhash-sketched into executor-local C++ hash tables, globally merged with map-reduce, and written back to HDFS as per-executor partition files.

## When to use

Use when reference genome collections are too large for single-workstation k-mer database construction and a YARN/HDFS cluster with Spark 2 is available, requiring a partitioned minhash database instead of one monolithic file.

## When NOT to use

- Reference genome collection is small enough for a single-workstation database — use the C++ AFS-MetaCache or muellan/metacache CPU build instead of the Spark pipeline
- No YARN/HDFS big-data cluster is available (MetaCacheSpark requires Hadoop ecosystem, Spark 2, and HDFS for both input and output)
- Goal is classification of reads rather than database construction — run in query mode (-m query) against an already built database

## Inputs

- Reference genome sequences stored in HDFS (FASTA)
- Taxonomy files stored in HDFS (path passed via -t)
- MetaCacheSpark-0.4.0.jar built from the cloned repository with Maven

## Outputs

- Partitioned reference database 'Database_16' in HDFS at the user home, consisting of executor-local minhash hash table files (one partition per executor, items of each reference sequence confined to one table)

## How to apply

Clone the MetaCacheSpark repository and build it with Maven (mvn package) to produce target/MetaCacheSpark-0.4.0.jar; the project needs a YARN/HDFS cluster, Spark 2, Java 8, Maven 3, and a C++14 compiler since part of metacache's native C++ code and JNI glue is compiled into the jar. Upload reference genome sequences and taxonomy files to HDFS, then launch via spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark on YARN (e.g. --num-executors 16 --executor-memory 15G --driver-memory 20G) with build-mode parameters: -m build, -o (remove-overpopulated-features, recommended for large data to avoid classification-phase errors), -p 16 (partitions), and -t pointing to the taxonomy path in HDFS. Each executor minhash-sketches its share of sequences into executor-local C++ hash tables; distribution among executors uses Spark repartition (-e) or, by default, proportional-to-sequence-length assignment. A map-reduce merge then combines global key counts across executors and deletes overpopulated features (deletion threshold default 254), ensuring each reference sequence's items reside in exactly one hash table. Finally each executor-local hash table is written to HDFS at the user home under the given database name (e.g. Database_16).

## Related tools

- **MetaCacheSpark** (Primary tool: Spark wrapper around metacache's minhashing algorithm; distributes sequences to executors, builds executor-local C++ hash tables via JNI, merges and writes the partitioned database to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Execution engine on YARN: provides executor distribution of sequences, map-reduce global key-count merge, and HDFS I/O)
- **AFS-MetaCache** (C++ single-workstation counterpart producing equivalent minhash databases with reduced memory requirements; alternative when no cluster is needed) — https://muellan.github.io/metacache/afs.html
- **metacache** (Origin of the minhashing algorithm and native C++ hash-table code linked into MetaCacheSpark via JNI) — https://github.com/muellan/metacache

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark --master yarn --num-executors 16 --executor-memory 15G --driver-memory 20G MetaCacheSpark-0.4.0.jar -m build -o -p 16 -t /path/to/taxonomy/in/hdfs/ Database_16 /path/to/input/sequences/in/hdfs/
```

## Evaluation signals

- Database 'Database_16' exists in HDFS at the user home after the build job completes
- Database is loadable in query mode: spark-submit with -m query against Database_16 runs classification without errors
- Number of partitions written matches the -p value (e.g. 16 partitions for -p 16 --num-executors 16)
- Overpopulated features above the deletion threshold (default 254) have been removed when -o is set, avoiding classification-phase errors
- Each reference sequence's items reside in exactly one hash table partition (no duplication across executors)

## Limitations

- The -o remove-overpopulated-features option should be used for large datasets; omitting it can cause errors during the classification phase
- Default sequence distribution (by length, without -e) may imbalance partitions; -e Spark repartition is optional
- Requires the full Hadoop ecosystem (YARN, HDFS), Spark 2, Java 8, Maven 3, and a C++14 compiler; not runnable on a plain workstation
- MetaCacheSpark supports only build and query modes plus limited parameters compared to the standalone metacache

## Evidence

- [methods] MetaCacheSpark's build pipeline loads reference genome sequences from HDFS and distributes them proportionally among Spark executors, which then apply the minhashing algorithm into executor-local C++ hash tables.: [evidence span withheld — anchor status: not_found]
- [readme] Build example from README with 16 executors writing Database_16 to HDFS.: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS at the user home with the name *Database_16*."
- [readme] -o flag rationale for large-scale builds.: "**-o**: Uses the *remove-overpopulated-features*. As theoretically, in the case of using **MetaCacheSpark** the user is dealing with a lot of data, this option should be used"
- [abstract] Database partitioning reduces memory needs, the motivation for cluster construction.: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
