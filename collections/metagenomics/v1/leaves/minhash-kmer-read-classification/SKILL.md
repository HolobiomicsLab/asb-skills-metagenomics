---
name: minhash-kmer-read-classification
description: Use when shotgun reads (FASTA/FASTQ, short or long) from metagenomic
  or all-food-sequencing (AFS) samples must be taxonomically classified and quantified
  against massive multi-kingdom reference genome collections faster than alignment-based
  tools like BWA or MegaBLAST.
when_to_use_negative:
- The task requires base-level alignment of reads to reference sequences (use BWA,
  Bowtie2, or CUSHAW instead) — MetaCache only assigns reads to taxa via k-mer/minhash
  similarity
- The reference database and the classification executable were compiled with different
  data type settings (e.g. MC_TARGET_ID_TYPE or MC_KMER_TYPE) — the README warns a
  database can only be queried with the same variant that built it
- Highly accurate alignment-based quantification is required for novel or highly divergent
  organisms absent from reference genomes, where k-mer matching false positives matter
edam_operation: http://edamontology.org/operation_0367
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: AFS-MetaCache
  role: C++ workstation version of the minhashing-based classifier; builds partitioned
    databases (1/2/4 partitions) and classifies reads on a single machine
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark version for big data clusters; build mode and query mode run
    via spark-submit on YARN with HDFS-stored databases
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: MetaCache
  role: Original minhashing-based metagenomic classification system from which AFS-MetaCache
    and MetaCacheSpark derive
  repo: https://github.com/muellan/metacache
- name: Apache Spark
  role: Distributed engine used by MetaCacheSpark to parallelize database building
    and read classification across YARN executors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:55908d21271c284551aff1bb54a59e8136ffa899fe91ae5fd1ca02423a94d259
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCache
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0367
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# minhash-kmer-read-classification

## Summary

Classify metagenomic/food-sample sequencing reads against large reference genome databases using MetaCache's alignment-free, minhashing-subsampled k-mer matching, implemented either as the C++ workstation tool (AFS-MetaCache) or as the Apache Spark cluster tool (MetaCacheSpark).

## When to use

Use when shotgun reads (FASTA/FASTQ, short or long) from metagenomic or all-food-sequencing (AFS) samples must be taxonomically classified and quantified against massive multi-kingdom reference genome collections faster than alignment-based tools like BWA or MegaBLAST.

## When NOT to use

- The task requires base-level alignment of reads to reference sequences (use BWA, Bowtie2, or CUSHAW instead) — MetaCache only assigns reads to taxa via k-mer/minhash similarity
- The reference database and the classification executable were compiled with different data type settings (e.g. MC_TARGET_ID_TYPE or MC_KMER_TYPE) — the README warns a database can only be queried with the same variant that built it
- Highly accurate alignment-based quantification is required for novel or highly divergent organisms absent from reference genomes, where k-mer matching false positives matter

## Inputs

- Shotgun sequencing reads in FASTA or FASTQ (short reads ~100-130bp or long reads, optionally paired-end)
- Reference genome collection (multi-kingdom eukaryotic, bacterial, viral sequences) or a pre-built MetaCache database (e.g. AFS31RS90 or NCBI RefSeq-based)

## Outputs

- Taxonomic classification of each read mapping to its most likely taxon of origin (per-read taxon assignments in results.txt or HDFS)
- Abundance estimation of organisms at a selectable taxonomic level (species composition of the sample)

## How to apply

First build a reference database from reference genomes in build mode ('-m build'), employing minhashing to subsample k-mers within local windows so stored k-mers are reduced by an order of magnitude, and enabling the remove-overpopulated-features option ('-o') to prune features; for databases too large for RAM, use the partitioning scheme (e.g. 1/2/4 partitions in AFS-MetaCache, or '-p' partitions per executor in MetaCacheSpark, stored in HDFS). Then classify reads in query mode ('-m query') with MetaCacheSpark ('-n' threads per executor, e.g. 4 threads with 16 executors) or AFS-MetaCache, selecting a query_mode ('precise', 'threshold', 'fast', or 'very_fast') and optionally '-g hits_greater_than' to filter candidate features. Optionally enable abundance estimation at a selectable taxonomic level with '-a abundance_per' and paired-read handling with '-r'. Classified results are written to a text output file (metacache) or HDFS (MetaCacheSpark); validation compares false-positive rate and quantification accuracy against CLARK, Kraken2, and Kraken2+Bracken.

## Related tools

- **AFS-MetaCache** (C++ workstation version of the minhashing-based classifier; builds partitioned databases (1/2/4 partitions) and classifies reads on a single machine) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark version for big data clusters; build mode and query mode run via spark-submit on YARN with HDFS-stored databases) — https://github.com/jmabuin/MetaCacheSpark
- **MetaCache** (Original minhashing-based metagenomic classification system from which AFS-MetaCache and MetaCacheSpark derive) — https://github.com/muellan/metacache
- **Apache Spark** (Distributed engine used by MetaCacheSpark to parallelize database building and read classification across YARN executors)

## Examples

```
./metacache query refseq myReads.fa -out results.txt
```

## Evaluation signals

- Build time and peak main memory scale as expected with partition count: for AFS31RS90, 4 partitions reduce peak memory from 135 GB to 52 GB while build time rises only from 3h 10m to 3h 45m (paper Table 8)
- Query throughput on the KAL_D read set is recorded in MR/m for each configuration (e.g. 7.2 MR/m unpartitioned vs 2.5 MR/m with 4 partitions)
- False-positive rate and quantification accuracy of classifications are superior to CLARK, Kraken2, and Kraken2+Bracken on the same benchmark data
- Every read in the input FASTA/FASTQ receives a taxon assignment in the output file/HDFS result set; paired-end mode ('-r'/'-pairseq') consumes both mates consistently
- Abundance estimates are produced at the requested taxonomic level ('-a abundance_per') and remove-overpopulated-features ('-o') has pruned overrepresented features from the database

## Limitations

- Partitioning trades speed for memory: querying a 4-partition AFS31RS90 database drops throughput from 7.2 to 2.5 MR/m
- Databases can only be queried by a MetaCache build with identical data-type compilation settings; mixing 32-bit and 64-bit compiled executables may be problematic due to endianness/type-width differences
- Default data types cap k-mer length at 16 and targets at ~4.2 billion; longer k-mers require recompilation with MC_KMER_TYPE=uint64_t
- Paper omits hardware specs of the benchmark cluster, software versions of compared tools (Kraken2, CLARK, Bracken), and reference genome identifiers, complicating exact reproduction

## Evidence

- [readme] It uses locality sensitive hashing to quickly identify candidate regions within one or multiple reference genomes.: [evidence span withheld — anchor status: external_doc]
- [intro] We employ a big data technique called minhashing to subsample k-mers: "We employ a big data technique called minhashing to subsample k-mers in an intelligent way, thereby reducing the amount of stored k-mers by an order-of-magnitude"
- [intro] can achieve high read classification accuracy while being around three orders-of-magnitude faster than the alignment tool MegaBLAST: "can achieve high read classification accuracy while being around three orders-of-magnitude faster than the alignment tool MegaBLAST"
- [readme] -m,--mode <arg>  Operation mode to use with MetaCacheSpark. Available options are: build, query: " -m,--mode <arg>                      Operation mode to use with
                                          MetaCacheSpark.
                                          Available options are: build,"
- [readme] -q,--query_mode <arg>  Mode selected for query. Available options are: precise, threshold, fast, very_fast.: " -q,--query_mode <arg>                [QUERY] Mode selected for query.
                                          Available options are: precise,
                                          threshold,"
