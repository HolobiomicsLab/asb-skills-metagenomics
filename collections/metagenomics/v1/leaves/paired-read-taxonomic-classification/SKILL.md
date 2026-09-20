---
name: paired-read-taxonomic-classification
description: Use when paired-end read pairs (e.g. KAL_D, 26,114K read pairs) must
  be taxonomically assigned against multi-kingdom reference genome databases, especially
  when benchmarking MetaCache/AFS-MetaCache, CLARK, or Kraken2+Bracken classification
  quality or runtime.
when_to_use_negative:
- Input is single-end reads only and paired-read mode (-r) cannot be exploited
- The task requires alignment-based read placement (e.g. BWA/Bowtie2 in the original
  AFS pipeline) rather than alignment-free k-mer classification
- The database was built with different MetaCache data-type compilation settings than
  the querying binary (databases only work with the same variant)
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier run in query mode with paired reads (-r) and threshold
    query mode against AFS reference databases
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark classifier executed with 16 executors and 4 threads per executor
    for paired-read classification on a big data cluster; database stored in HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Kraken2+Bracken
  role: Comparison classifier for accuracy (false-positive rate, quantification accuracy)
    and runtime benchmarking
- name: CLARK
  role: Comparison classifier; exceeded 512 GB workstation memory when building the
    AFS31RS90 database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:5772e837359c4a8b4affc3c147737329fbca058d4da5eb2d4b6f47b7e6dae79c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# paired-read-taxonomic-classification

## Summary

Classifies paired-end sequencing reads against large AFS reference databases (AFS20, AFS31, AFS20RS90, AFS31RS90) using minhashing-based metagenomic classifiers in paired-read mode with threshold query logic, for detection and quantification of food sample composition.

## When to use

Use when paired-end read pairs (e.g. KAL_D, 26,114K read pairs) must be taxonomically assigned against multi-kingdom reference genome databases, especially when benchmarking MetaCache/AFS-MetaCache, CLARK, or Kraken2+Bracken classification quality or runtime.

## When NOT to use

- Input is single-end reads only and paired-read mode (-r) cannot be exploited
- The task requires alignment-based read placement (e.g. BWA/Bowtie2 in the original AFS pipeline) rather than alignment-free k-mer classification
- The database was built with different MetaCache data-type compilation settings than the querying binary (databases only work with the same variant)

## Inputs

- Paired-end FASTQ/FASTA read pairs (e.g. KAL_D, 26,114K read pairs)
- Pre-built classification database (AFS20, AFS31, AFS20RS90, or AFS31RS90) in MetaCache/HDFS format
- Taxonomy files associated with the reference genome collection

## Outputs

- Per-read taxonomic assignments to reference taxa
- Abundance estimates at a selectable taxonomic level (via abundance_per option)
- Query wall-clock time, peak query memory, and speed in MR/m for benchmark tables like Table 7

## How to apply

Build or load an AFS reference database (enabling the remove-overpopulated-features option, -o, at build time to guard against classification-phase errors), then run the classifier in query mode with paired-read support enabled (-r / --paired_reads) and the threshold query mode, which calculates a threshold value according to classification parameters. For MetaCacheSpark on a cluster, execute with 16 executors and 4 threads per executor (--num-executors 16 --executor-cores 4), storing databases in HDFS. For the C++ AFS-MetaCache on a workstation, invoke metacache query with paired files or interleaved paired sequences. Measure wall-clock time and peak memory (e.g. /usr/bin/time -v) and convert query time to speed in millions of reads per minute (MR/m).

## Related tools

- **AFS-MetaCache** (C++ workstation classifier run in query mode with paired reads (-r) and threshold query mode against AFS reference databases) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark classifier executed with 16 executors and 4 threads per executor for paired-read classification on a big data cluster; database stored in HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Kraken2+Bracken** (Comparison classifier for accuracy (false-positive rate, quantification accuracy) and runtime benchmarking)
- **CLARK** (Comparison classifier; exceeded 512 GB workstation memory when building the AFS31RS90 database)

## Examples

```
./metacache query refseq myReads1.fa myReads2.fa -pairfiles -out results.txt
```

## Evaluation signals

- Query job completes over all 26,114K KAL_D read pairs with paired-read mode (-r) and threshold query mode active
- Measured query speed falls within the order-of-magnitude reported in the paper (MetaCache-classified reads per minute vs. MegaBLAST-scale alignment runtimes)
- False-positive rate and quantification accuracy of AFS-MetaCache/MetaCacheSpark are equal to or better than CLARK, Kraken2, and Kraken2+Bracken as reported
- Benchmark table (Table 7 layout) contains all 4 tools x 4 databases rows with query time, query speed (MR/m), and query memory populated
- MetaCacheSpark query executed with 16 executors and 4 threads per executor as in the paper's example configuration

## Limitations

- CLARK exceeded 512 GB workstation RAM on the AFS31RS90 database (reported as '-'), so comparisons on the largest database may be incomplete
- MetaCacheSpark only supports build and query modes; add, info, and annotate modes are listed but classification requires threshold/precise/fast/very_fast parameter tuning
- No software version numbers are given for Kraken2, CLARK, Bracken, or Spark, limiting reproducibility of the benchmark
- Databases built with one variant (data-type settings, endianness) cannot be queried with a different MetaCache binary

## Evidence

- [methods] Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor.: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [readme] paired_reads parameter: "-r,--paired_reads                    [QUERY] Use paired reads in the"
- [readme] query mode options: "Available options are: precise,
                                          threshold, fast, very_fast. Default:
                                          threshold"
- [abstract] superior accuracy finding: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [methods] build-time guard option: "* **-o**: Uses the *remove-overpopulated-features*."
