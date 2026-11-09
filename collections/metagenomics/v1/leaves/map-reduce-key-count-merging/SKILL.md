---
name: map-reduce-key-count-merging
description: Use when building a distributed metagenomics reference database on Spark
  from minhash sketches stored in per-executor hash tables and the counts must be
  merged globally before writing to HDFS.
when_to_use_negative:
- Running on a single workstation — use the AFS-MetaCache C++ version instead, which
  does not need a map-reduce merge across executors
- The database is already built and stored in HDFS — proceed directly to query mode
  (-m query) rather than re-merging
- Input is raw sequencing reads from a metagenomic sample (FASTA/FASTQ) rather than
  reference genome sequences — this merge step belongs to database construction, not
  classification
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaCacheSpark
  role: Implements the build pipeline whose map-reduce key-count merge combines executor-local
    hash tables and removes overpopulated features before writing the partitioned
    database to HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Execution engine (on YARN with HDFS) providing the executors, driver, and
    map-reduce machinery across which key counts are aggregated
- name: metacache
  role: Provides the underlying minhashing algorithm and C++ hash table code used
    by MetaCacheSpark; its single-node version does not require this distributed merge
  repo: https://github.com/muellan/metacache
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:22c500a695ed27ce5bcdbd58ff10b5340e835ea6c37f94e547ce6eb3ae47dc30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  - metacache
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# map-reduce-key-count-merging

## Summary

The MetaCacheSpark build phase in which executor-local minhash hash tables are globally merged via a map-reduce key-count operation across the Spark cluster, removing overpopulated k-mer features so the partitioned reference database remains loadable for classification.

## When to use

Use when building a distributed metagenomics reference database on Spark from minhash sketches stored in per-executor hash tables and the counts must be merged globally before writing to HDFS.

## When NOT to use

- Running on a single workstation — use the AFS-MetaCache C++ version instead, which does not need a map-reduce merge across executors
- The database is already built and stored in HDFS — proceed directly to query mode (-m query) rather than re-merging
- Input is raw sequencing reads from a metagenomic sample (FASTA/FASTQ) rather than reference genome sequences — this merge step belongs to database construction, not classification

## Inputs

- Per-executor C++ hash tables produced by executor-local minhash sketching of reference genome sequences
- Reference genome sequences previously loaded from HDFS and proportionally distributed among executors (via -e Spark repartition or by sequence length)
- Taxonomy files stored in HDFS (passed via -t)

## Outputs

- A partitioned reference genome database in HDFS at the user home (e.g. Database_16), composed of executor-local hash table files
- A merged key-count table with overpopulated features removed (counts above the deletion threshold, default 254)

## How to apply

After each Spark executor has independently minhash-sketched its share of HDFS-hosted reference sequences into a local C++ hash table, perform a map-reduce global key-count merge across the executors (launched with 16 executors, 15G executor-memory, 20G driver-memory on YARN). During the merge, delete overpopulated features whose counts exceed the deletion threshold (default 254) by enabling the -o/--remove_overpopulated_features build flag — the README notes this is essential with large collections to avoid errors during the subsequent classification phase. The merge must ensure that each reference sequence's items reside in exactly one hash table, keeping the database partitioned. Finally, write each executor-local hash table to HDFS under a single database name (e.g. Database_16), validated by loading it in query mode (-m query).

## Related tools

- **MetaCacheSpark** (Implements the build pipeline whose map-reduce key-count merge combines executor-local hash tables and removes overpopulated features before writing the partitioned database to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Execution engine (on YARN with HDFS) providing the executors, driver, and map-reduce machinery across which key counts are aggregated)
- **metacache** (Provides the underlying minhashing algorithm and C++ hash table code used by MetaCacheSpark; its single-node version does not require this distributed merge) — https://github.com/muellan/metacache

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark --conf spark.network.timeout=10000000 --num-executors 16 --master yarn --executor-memory 15G --driver-memory 20G MetaCacheSpark-0.4.0.jar -m build -o -p 16 -t /path/to/taxonomy/in/hdfs/ Database_16 /path/to/input/sequences/in/hdfs/
```

## Evaluation signals

- The merged database (e.g. Database_16) exists in HDFS at the user home and is loadable in query mode via spark-submit with -m query
- No feature in the merged key-count table exceeds the deletion threshold (default 254) when the -o remove-overpopulated-features flag was enabled
- Each reference sequence's items reside in exactly one executor-local hash table after the merge (no cross-partition duplication)
- The number of partitions written to HDFS matches the -p 16 setting used with 16 executors
- A subsequent classification run on the merged database completes without the errors that the README warns about when overpopulated features are not removed

## Limitations

- Without the -o remove-overpopulated-features option, large collections can cause errors during the classification phase
- Requires a full Hadoop ecosystem (YARN + HDFS) cluster with Spark 2 and Java 8; not runnable as a plain single-node job
- Sequence distribution matters: without the -e Spark repartition flag, sequences are distributed by their length, which affects how the executor-local tables are formed before merging

## Evidence

- [methods] Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [readme] remove-overpopulated-features rationale: "Uses the *remove-overpopulated-features*. As theoretically, in the case of using **MetaCacheSpark** the user is dealing with a lot of data, this option should be used in order to avoid errors"
- [abstract] partitioning rationale: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [readme] sequence distribution among executors: "Indicates if the input reads are distributed among the cluster by using a Spark *repartition* or not. If not, sequences are distributed according their length."
