---
name: minhash-kmer-sketching
description: Use when building a large reference genome classification database (e.g.
  multi-kingdom food or RefSeq-scale collections) where storing all k-mers would exhaust
  memory and a minhash sketch must be computed into partitioned hash tables.
when_to_use_negative:
- Input is already a built classification database or a query read set (use query
  mode, not sketch construction)
- The reference collection is small enough to fit all k-mers in memory on one machine
  with the plain C++ metacache, where a distributed Spark sketch adds unnecessary
  complexity
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MetaCacheSpark
  role: Distributed implementation that minhash-sketches HDFS-hosted reference genomes
    into executor-local C++ hash tables on a Spark/YARN cluster
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: AFS-MetaCache
  role: C++ workstation implementation of the same minhashing-based database construction
    with partitioning for reduced memory
  repo: https://muellan.github.io/metacache/afs.html
- name: metacache
  role: Original CPU/GPU MetaCache classification system supplying the minhashing
    algorithm and native C++ hash-table code reused by MetaCacheSpark via JNI
  repo: https://github.com/muellan/metacache
- name: Apache Spark
  role: Execution engine distributing reference sequences among executors and performing
    the map-reduce global key-count merge of sketched features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:7d209ec4a37490a5c4c05edd1cf4110cb0cb5924677610ba83b7be647326d45b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  - metacache
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# minhash-kmer-sketching

## Summary

Subsample k-mers from reference genomes using locality-sensitive minhashing so that only an order-of-magnitude fewer k-mers are stored while read classification accuracy is retained. This is the core indexing technique behind MetaCache, AFS-MetaCache, and MetaCacheSpark.

## When to use

Use when building a large reference genome classification database (e.g. multi-kingdom food or RefSeq-scale collections) where storing all k-mers would exhaust memory and a minhash sketch must be computed into partitioned hash tables.

## When NOT to use

- Input is already a built classification database or a query read set (use query mode, not sketch construction)
- The reference collection is small enough to fit all k-mers in memory on one machine with the plain C++ metacache, where a distributed Spark sketch adds unnecessary complexity

## Inputs

- Reference genome sequences in FASTA/FASTQ format (uploaded to HDFS for MetaCacheSpark)
- Taxonomy files placed in HDFS (passed via -t)
- Configuration parameters (e.g. src/main/resources/config.properties, optionally passed with -c)

## Outputs

- Partitioned minhash-sketch database of executor-local C++ hash tables stored in HDFS (e.g. Database_16), one hash table per partition with each reference sequence's items in exactly one table
- Feature counts after global key-count merge with overpopulated features (above threshold 254) removed

## How to apply

Load reference genome sequences (FASTA/FASTQ from HDFS for MetaCacheSpark) and distribute them across workers, either via Spark repartition (-e) or proportionally by sequence length. Each worker applies the MetaCache minhashing algorithm, which subsamples k-mers intelligently within local windows, sketching its sequences into a local C++ hash table. Run the map-reduce global key-count merge across executors, and apply the remove-overpopulated-features option (-o) to delete features above the deletion threshold (default 254) so each reference sequence's items reside in exactly one hash table. Finally write each executor-local hash table to HDFS as a partitioned database (e.g. Database_16 with -p 16), which is the sketch-based structure later queried for classification.

## Related tools

- **MetaCacheSpark** (Distributed implementation that minhash-sketches HDFS-hosted reference genomes into executor-local C++ hash tables on a Spark/YARN cluster) — https://github.com/jmabuin/MetaCacheSpark
- **AFS-MetaCache** (C++ workstation implementation of the same minhashing-based database construction with partitioning for reduced memory) — https://muellan.github.io/metacache/afs.html
- **metacache** (Original CPU/GPU MetaCache classification system supplying the minhashing algorithm and native C++ hash-table code reused by MetaCacheSpark via JNI) — https://github.com/muellan/metacache
- **Apache Spark** (Execution engine distributing reference sequences among executors and performing the map-reduce global key-count merge of sketched features)

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark --master yarn --num-executors 16 --executor-memory 15G --driver-memory 20G MetaCacheSpark-0.4.0.jar -m build -o -p 16 -t /path/to/taxonomy/in/hdfs/ Database_16 /path/to/input/sequences/in/hdfs/
```

## Evaluation signals

- The database (e.g. Database_16) exists in HDFS at the user home and is loadable by re-running spark-submit with -m query against it
- Each reference sequence's items reside in exactly one hash table after the map-reduce global key-count merge
- Overpopulated features above the deletion threshold (default 254) have been removed when -o is set
- Stored k-mer count is reduced by roughly an order-of-magnitude relative to exhaustive k-mer indexing, per the paper's minhashing claim
- Build completes with the specified partition count (-p 16 producing 16 executor-local hash tables)

## Limitations

- Remove-overpopulated-features (-o) should be used with large inputs to avoid classification-phase errors, since heavily shared features can overpopulate executor-local tables
- Sequence distribution strategy matters: without -e, sequences are distributed by length rather than via Spark repartition, affecting per-executor load
- Minhashing subsamples k-mers, so sketch coverage trades database size against sensitivity compared to full k-mer indexes

## Evidence

- [intro] We employ a big data technique called minhashing to subsample k-mers in an intelligent way, thereby reducing the amount of stored k-mers by an order-of-magnitude: "We employ a big data technique called minhashing to subsample k-mers in an intelligent way, thereby reducing the amount of stored k-mers by an order-of-magnitude"
- [intro] we combine the minhashing algorithm used by MetaCache with efficient partitioning schemes. This allows us to employ databases that index large collections of reference genomes efficiently: "we combine the minhashing algorithm used by MetaCache with efficient partitioning schemes. This allows us to employ databases that index large collections of reference genomes efficiently"
- [readme] -o,--remove_overpopulated_features   [BUILD] Uses remove overpopulated features when building.: "-o,--remove_overpopulated_features   [BUILD] Uses remove overpopulated features when building."
- [readme] -e: Indicates if the input reads are distributed among the cluster by using a Spark repartition or not. If not, sequences are distributed according their length.: "-e: Indicates if the input reads are distributed among the cluster by using a Spark *repartition* or not. If not, sequences are distributed according their length."
- [readme] -o: Uses the remove-overpopulated-features. As theoretically, in the case of using MetaCacheSpark the user is dealing with a lot of data, this option should be used: "As theoretically, in the case of using **MetaCacheSpark** the user is dealing with a lot of data, this option should be used in order to avoid errors during the classification phase."
- [readme] It uses locality sensitive hashing to quickly identify candidate regions within one or multiple reference genomes.: [evidence span withheld — anchor status: external_doc]
