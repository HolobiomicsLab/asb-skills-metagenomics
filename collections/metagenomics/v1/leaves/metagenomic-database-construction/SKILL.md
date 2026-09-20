---
name: metagenomic-database-construction
description: Use when constructing a k-mer/minhash classification database from large
  multi-kingdom reference genome collections where full-database build exceeds workstation
  memory, or when benchmarking build time and peak RAM across partition/executor counts.
when_to_use_negative:
- Input is already a built MetaCache database — use query mode (-m query) instead
  of rebuilding
- The reference collection is small enough to fit in workstation RAM without partitioning
  and no build-time/memory benchmark is needed
- Reads must be aligned for downstream variant or assembly analysis — use an alignment
  tool (BWA, Bowtie2) rather than a k-mer classification database
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation used to build the reference database with 1,
    2, or 4 partitions and the remove-overpopulated-features option
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark implementation run via spark-submit in build mode (-m build) on a YARN/HDFS
    cluster, scaling across 8-64 executors
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Big-data engine underlying MetaCacheSpark database construction on the cluster
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: MetaCache
  role: Original minhashing classification system whose algorithm is extended and
    packaged as AFS-MetaCache/MetaCacheSpark
  repo: https://github.com/muellan/metacache
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:f3024ab561c365bf8d94dd289259910f250bce4bd5616eeb36ab7169969afd87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - BWA
  - MetaCache
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metagenomic-database-construction

## Summary

Build minhashing-based metagenomic classification databases (MetaCache/AFS-MetaCache/MetaCacheSpark) from large reference genome collections, using database partitioning and remove-overpopulated-features filtering to keep build time and peak memory tractable for all-food-sequencing style reference sets (e.g. the 135 GB-class AFS31RS90 database).

## When to use

Use when constructing a k-mer/minhash classification database from large multi-kingdom reference genome collections where full-database build exceeds workstation memory, or when benchmarking build time and peak RAM across partition/executor counts.

## When NOT to use

- Input is already a built MetaCache database — use query mode (-m query) instead of rebuilding
- The reference collection is small enough to fit in workstation RAM without partitioning and no build-time/memory benchmark is needed
- Reads must be aligned for downstream variant or assembly analysis — use an alignment tool (BWA, Bowtie2) rather than a k-mer classification database

## Inputs

- Collection of reference genome sequences (FASTA) spanning food-relevant eukaryotic and microbial taxa (e.g. AFS10, AFS31RS90 genome collections)
- Taxonomy/mapping information for the reference targets
- Spark cluster configuration (number of executors, executor/driver memory) for MetaCacheSpark builds
- Optional configuration file (src/main/resources/config.properties) passed via -c

## Outputs

- MetaCache classification database (built for subsequent '-m query' classification runs)
- For MetaCacheSpark: database stored in HDFS
- Build-time and peak main-memory measurements per configuration (e.g. results_table.csv for Table 8 reproduction)

## How to apply

Choose the implementation that matches the compute environment: AFS-MetaCache (C++, workstation) or MetaCacheSpark (spark-submit on a YARN-managed cluster storing the database in HDFS). Build the database with the '-m build' mode and enable the remove-overpopulated-features option (-o), which filters target references based on sequence coverage and was used in the paper's build workflow. If peak memory is limiting, apply the database partitioning scheme — e.g. splitting the AFS31RS90 database into 1, 2, or 4 partitions with AFS-MetaCache, or distributing sequences over executors (8/16/32/64, e.g. 15G executor memory, 20G driver memory) with MetaCacheSpark — trading a modest build-time increase (3h10m → 3h45m at 4 partitions) for lower peak RAM (135 GB → 52 GB). Build MetaCacheSpark from source with Maven ('mvn package') producing MetaCacheSpark-0.4.0.jar, requiring Spark 2, Java 8, Maven 3, and a C++14 compiler for the JNI native code. Record build time and peak main memory per configuration so scaling behaviour can be validated against reported results (e.g. Table 8).

## Related tools

- **AFS-MetaCache** (C++ workstation implementation used to build the reference database with 1, 2, or 4 partitions and the remove-overpopulated-features option) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark implementation run via spark-submit in build mode (-m build) on a YARN/HDFS cluster, scaling across 8-64 executors) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Big-data engine underlying MetaCacheSpark database construction on the cluster) — https://github.com/jmabuin/MetaCacheSpark
- **MetaCache** (Original minhashing classification system whose algorithm is extended and packaged as AFS-MetaCache/MetaCacheSpark) — https://github.com/muellan/metacache

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -p 16
```

## Evaluation signals

- Build succeeds for all planned configurations (AFS-MetaCache 1/2/4 partitions; MetaCacheSpark 8/16/32/64 executors) and a queryable database is produced/stored in HDFS
- Peak main memory decreases monotonically with partition/executor count, matching reported scaling (e.g. 135 GB at 1 partition → 52 GB at 4 partitions for AFS31RS90)
- Build time increases only modestly with partitioning (3h 10m → 3h 45m for 1 → 4 partitions), within the expected range
- The built database yields expected query speeds in '-m query' mode (e.g. 7.2 MR/m at 1 partition dropping to 2.5 MR/m at 4 partitions on KAL_D)
- Remove-overpopulated-features (-o) was enabled during build as in the paper's workflow, confirmed from the command logs

## Limitations

- Partitioning reduces peak memory but lowers query speed (7.2 → 2.5 MR/m for AFS31RS90 split into 4 partitions)
- MetaCacheSpark only supports build and query modes and requires a Hadoop ecosystem with YARN and HDFS, Spark 2, Java 8, Maven 3, and a C++14 compiler
- Databases can only be queried with the same MetaCache variant/data-type build settings used to construct them; platform endianness/width differences can break database compatibility
- Paper does not state reference genome versions/identifiers or cluster hardware specifics, complicating exact reproduction

## Evidence

- [abstract] Database partitioning allows processing massive reference genome collections with reduced memory on a workstation or Spark cluster.: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [methods] MetaCacheSpark build is run with spark-submit in build mode and stored in HDFS.: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [methods] The remove-overpopulated-features option is used during database construction.: "* **-o**: Uses the *remove-overpopulated-features*."
- [readme] MetaCacheSpark is built from source with Maven producing the launchable jar.: "git clone https://github.com/jmabuin/MetaCacheSpark.git
	cd MetaCacheSpark
	mvn package"
- [readme] The -o flag enables remove overpopulated features in build mode per the CLI docs.: "-o,--remove_overpopulated_features   [BUILD] Uses remove overpopulated features when building."
