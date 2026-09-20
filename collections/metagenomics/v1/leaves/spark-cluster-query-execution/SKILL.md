---
name: spark-cluster-query-execution
description: Use when classifying large metagenomic/food-sequencing read sets (e.g.
  paired-end FASTQ/FASTA in HDFS) against a minhashing reference database on a Spark/YARN
  cluster instead of a single workstation.
when_to_use_negative:
- When the reference database and read set fit comfortably in a large-RAM workstation
  — the C++ AFS-MetaCache build avoids cluster overhead
- When the classifier must be CLARK, Kraken2, or Kraken2+Bracken — they are not Spark-based
  and are run under their own default settings
- When only database construction is needed — build mode (-m build) is a separate
  operation with its own parameter set
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaCacheSpark
  role: Spark-based classifier executed in query mode on the cluster; provides threshold
    query mode, paired-read support, and abundance estimation
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Distributed execution engine used via spark-submit with 16 executors and 4
    threads per executor
- name: AFS-MetaCache
  role: C++ workstation counterpart used as the baseline comparison for query time,
    speed, and memory
  repo: https://muellan.github.io/metacache/afs.html
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:33e650d5a08d0af507eb569d0ce6e3fb55ff78df8f21c34b512f49b672d3391a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Hadoop
  - Apache Spark
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spark-cluster-query-execution

## Summary

Executes MetaCacheSpark's classification (query) mode on a Hadoop/Spark big-data cluster via spark-submit, using 16 executors with 4 threads per executor in threshold query mode to classify large read sets against databases stored in HDFS.

## When to use

Use when classifying large metagenomic/food-sequencing read sets (e.g. paired-end FASTQ/FASTA in HDFS) against a minhashing reference database on a Spark/YARN cluster instead of a single workstation.

## When NOT to use

- When the reference database and read set fit comfortably in a large-RAM workstation — the C++ AFS-MetaCache build avoids cluster overhead
- When the classifier must be CLARK, Kraken2, or Kraken2+Bracken — they are not Spark-based and are run under their own default settings
- When only database construction is needed — build mode (-m build) is a separate operation with its own parameter set

## Inputs

- Paired-end FASTQ/FASTA sequencing reads staged in HDFS (e.g. KAL_D, 26,114K read pairs)
- Pre-built MetaCacheSpark reference database stored in HDFS (e.g. AFS20, AFS31, AFS20RS90, AFS31RS90)
- Taxonomy files in HDFS
- config.properties parameter file (optional, via -c)

## Outputs

- Per-read classification/abundance output at the chosen taxonomic level
- Query wall-clock time and peak query memory measurements convertible to speed (MR/m)
- Rows for performance comparison tables (e.g. Table 7 metrics: query time, query speed, query memory)

## How to apply

Submit the MetaCacheSpark jar with spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark in query mode (-m query), pointing at a database stored in HDFS. Use the paper's benchmark configuration of 16 executors and 4 threads per executor (--num-executors 16 with 4 classification threads via -n), and set the query mode (-q threshold) which computes a candidate threshold value from configurable parameters. Enable paired reads (-r) when classifying paired-end data such as the KAL_D dataset (26,114K read pairs) and optionally enable abundance estimation at a selectable taxonomic level (-a). Record wall-clock time and peak memory (convert time to speed in millions of reads per minute, MR/m) so results can be compared against workstation classifiers like AFS-MetaCache, CLARK, and Kraken2.

## Related tools

- **MetaCacheSpark** (Spark-based classifier executed in query mode on the cluster; provides threshold query mode, paired-read support, and abundance estimation) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Distributed execution engine used via spark-submit with 16 executors and 4 threads per executor)
- **AFS-MetaCache** (C++ workstation counterpart used as the baseline comparison for query time, speed, and memory) — https://muellan.github.io/metacache/afs.html

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -r -q threshold --num-executors 16 -n 4 -d afs20_hdfs -q reads/
```

## Evaluation signals

- spark-submit completes with exit code 0 and the classification output is written for every partition (all reads classified)
- Query speed expressed in MR/m falls within the relative ranges reported in Table 7, with AFS-MetaCache/MetaCacheSpark fastest
- The --num-executors 16 and 4-threads-per-executor configuration matches the paper's example configuration
- Default query mode resolves to 'threshold' as documented in the MetaCacheSpark usage output
- Paired-read counts reconcile with input (e.g. 26,114K read pairs processed for KAL_D)

## Limitations

- MetaCacheSpark requires a Big Data cluster with YARN and HDFS, Spark 2, and Java 8; it only supports build and query modes plus add/info/annotate
- Query output depends on candidate filtering mode (precise, threshold, fast, very_fast); very_fast limits candidates to those equal or less than hits_greater_than, which can affect accuracy
- Hardware specifications of the big data cluster used for the paper's benchmarks are not provided, so absolute times may not reproduce exactly

## Evidence

- [methods] Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor.: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [readme] Available options are: precise, threshold, fast, very_fast. Default: threshold: "Available options are: precise, threshold, fast, very_fast. Default:
                                          threshold"
- [readme] MetaCacheSpark is a tool that allows to use the metagenomics minhashing algorithm from metacache by means of a Big Data environment by using Apache Spark as engine.: "**MetaCacheSpark** is a tool that allows to use the metagenomics minhashing algorithm from [**metacache**][1] by means of a Big Data environment by using Apache Spark as engine."
- [readme] -a,--abundance_per <arg> [QUERY] Indicates if use the abundance estimation feature and at which level.: "-a,--abundance_per <arg>             [QUERY] Indicates if use the
                                          abundance estimation feature and at
                                          which level."
- [methods] threshold: A threshold value is calculated according different parameters.: "* *threshold*: A threshold value is calculated according different parameters."
