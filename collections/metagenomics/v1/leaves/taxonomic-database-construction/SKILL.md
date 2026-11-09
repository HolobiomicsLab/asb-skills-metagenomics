---
name: taxonomic-database-construction
description: Use when raw shotgun reads from complex biological samples (e.g., food)
  must be classified against a large multi-kingdom reference genome collection, and
  the k-mer/minhash database first has to be constructed at scale.
when_to_use_negative:
- The reference genomes are few and small enough to fit in a workstation's RAM — use
  the single-machine C++ AFS-MetaCache instead of the Spark build
- The database already exists (e.g., AFS20 is already built in HDFS) — go directly
  to classification/quantification
- An alignment-based AFS pipeline (BWA/Bowtie2/CUSHAW per reference genome) is required
  by the protocol rather than the alignment-free k-mer approach
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: MetaCacheSpark
  role: Builds the k-mer/minhash reference database on the Spark cluster in build
    mode with remove-overpopulated-features enabled; database stored in HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: MetaCache (metacache)
  role: Underlying minhashing classification algorithm and C++ workstation reference
    implementation; source of the AFS-MetaCache approach
  repo: https://github.com/muellan/metacache
- name: Apache Spark
  role: Distributed execution engine (spark-submit with YARN) used to run the build
    and query phases across executors
- name: Apache Hadoop
  role: HDFS stores the reference genomes, taxonomy files, and the constructed database;
    YARN manages cluster resources
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:3336c056699557fbcb21506c436fb3a5003bb9612ce99fd0b423385f7371e368
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
  - MetaCache (metacache)
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# taxonomic-database-construction

## Summary

Build a k-mer/minhash reference database (e.g., the AFS20 all-food-sequencing database) with MetaCacheSpark in build mode on a Spark/HDFS cluster, enabling remove-overpopulated-features to control database size, then use it for taxonomic classification and abundance quantification.

## When to use

Use when raw shotgun reads from complex biological samples (e.g., food) must be classified against a large multi-kingdom reference genome collection, and the k-mer/minhash database first has to be constructed at scale.

## When NOT to use

- The reference genomes are few and small enough to fit in a workstation's RAM — use the single-machine C++ AFS-MetaCache instead of the Spark build
- The database already exists (e.g., AFS20 is already built in HDFS) — go directly to classification/quantification
- An alignment-based AFS pipeline (BWA/Bowtie2/CUSHAW per reference genome) is required by the protocol rather than the alignment-free k-mer approach

## Inputs

- Collection of reference genome sequences (FASTA) spanning eukaryotic and bacterial taxa (e.g., the AFS20 database)
- NCBI taxonomy tree files placed on HDFS (passed with -t)
- YARN/Hadoop cluster with Spark 2, Java 8, and the built MetaCacheSpark-0.4.0.jar

## Outputs

- MetaCache k-mer/minhash reference database stored in HDFS, usable by MetaCacheSpark query mode
- Build statistics/log of database construction time and memory consumption

## How to apply

Clone and package MetaCacheSpark (mvn package) to obtain MetaCacheSpark-0.4.0.jar, then launch spark-submit with the MetaCacheSpark class in build mode (-m build) using 16 executors so the database is stored in HDFS. Pass the reference genome FASTA collection as build input, the HDFS taxonomy path with -t, and enable the remove-overpopulated-features option (-o) to prune features whose occurrence count is too high, which reduces stored k-mers (minhashing already subsamples by an order of magnitude). Optionally use --repartition (-e) and set the number of partitions (-p) for load balancing across executors. The rationale is that efficient database partitioning lets the AFS20-scale multi-kingdom genome collection fit within reduced peak memory on a single node or cluster. Validate by checking the database is written to HDFS and by confirming downstream query-mode classification (default threshold query mode, -m query) runs against it.

## Related tools

- **MetaCacheSpark** (Builds the k-mer/minhash reference database on the Spark cluster in build mode with remove-overpopulated-features enabled; database stored in HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **MetaCache (metacache)** (Underlying minhashing classification algorithm and C++ workstation reference implementation; source of the AFS-MetaCache approach) — https://github.com/muellan/metacache
- **Apache Spark** (Distributed execution engine (spark-submit with YARN) used to run the build and query phases across executors)
- **Apache Hadoop** (HDFS stores the reference genomes, taxonomy files, and the constructed database; YARN manages cluster resources)

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -t hdfs:///taxonomy <reference_fasta_on_HDFS>
```

## Evaluation signals

- Database build completes via spark-submit in build mode (-m build) and the resulting database is present in HDFS
- Remove-overpopulated-features (-o) was enabled — the flag appears in the build command — so stored feature counts are reduced
- Taxonomy files were supplied with -t and are readable from HDFS by subsequent query runs
- Subsequent query-mode classification of the Kal A-E and KLyo A-D calibrator samples succeeds, confirming database usability
- Build time and peak memory are consistent with the paper's claim that AFS-MetaCache/MetaCacheSpark are fastest in database construction times with reduced memory

## Limitations

- MetaCacheSpark only supports build and query modes (plus info/annotate) — other workflows need the C++ version
- No software version numbers are given for the benchmarked stack (Spark, Hadoop), and cluster hardware specifications are not provided in the paper
- No specific versions or accession identifiers of the reference genomes used for AFS20 construction are stated
- Databases must be queried with the same MetaCache data-type variant that built them; endianness/compiler-width differences can make databases non-portable

## Evidence

- [methods] Build AFS20 with MetaCacheSpark build mode and overpopulated-feature removal: "Execute **MetaCacheSpark** for building with 16 executors. The database will be stored in HDFS"
- [methods] remove-overpopulated-features option reduces feature overload: "* **-o**: Uses the *remove-overpopulated-features*."
- [abstract] Rationale: database partitioning enables large reference collections with reduced memory: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [readme] MetaCacheSpark purpose and Spark-based cluster operation: "**MetaCacheSpark** is a tool that allows to use the metagenomics minhashing algorithm from [**metacache**][1] by means of a Big Data environment by using Apache Spark as engine."
- [readme] Build invocation and jar artifact: "This will create the *target* folder, which will contain the *jar* file needed to run **MetaCacheSpark**: **MetaCacheSpark-0.4.0.jar** - jar file to launch with Spark."
