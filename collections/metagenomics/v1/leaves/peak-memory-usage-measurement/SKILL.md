---
name: peak-memory-usage-measurement
description: Use when benchmarking reference-database build or read-classification
  runs of k-mer based classifiers (e.g. on AFS20/AFS31/AFS20RS90/AFS31RS90 databases)
  and you need a comparable peak-RAM metric per tool/database combination.
when_to_use_negative:
- When measuring accuracy metrics (false-positive rate, quantification accuracy) rather
  than resource consumption — use classification output comparison instead
- When benchmarking on a Spark cluster where per-executor memory accounting differs
  from single-workstation resident-set measurement
- When the tool cannot complete on available RAM (as with CLARK on the 512 GB workstation
  for AFS31RS90) — record it as '-' rather than a partial peak
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier whose build/query runs are profiled for peak memory;
    fastest and lowest-memory constructor (3h 10m, 135 GB on AFS31RS90)
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark-based classifier executed with 16 executors and 4 threads per executor;
    its build and query memory consumption is measured alongside runtime
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Comparison classifier whose AFS31RS90 build exceeded the 512 GB workstation
    memory, recorded as '-' in the benchmark table
- name: Kraken2
  role: Comparison classifier; peak memory recorded as 134 GB for the AFS31RS90 build
- name: Kraken2+Bracken
  role: Comparison classifier with abundance estimation; peak build memory recorded
    as 329 GB for AFS31RS90
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:1ad7d00e64aa0fc3369cd51bf8e3706d6ba662e2a9771cec64a5d645090b817b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# peak-memory-usage-measurement

## Summary

Benchmarking technique that records the maximum resident memory consumed during database construction and classification runs of metagenomic classifiers (AFS-MetaCache, MetaCacheSpark, CLARK, Kraken2, Kraken2+Bracken), enabling fair comparison of memory footprints as reported in the paper's performance Table 7.

## When to use

Use when benchmarking reference-database build or read-classification runs of k-mer based classifiers (e.g. on AFS20/AFS31/AFS20RS90/AFS31RS90 databases) and you need a comparable peak-RAM metric per tool/database combination.

## When NOT to use

- When measuring accuracy metrics (false-positive rate, quantification accuracy) rather than resource consumption — use classification output comparison instead
- When benchmarking on a Spark cluster where per-executor memory accounting differs from single-workstation resident-set measurement
- When the tool cannot complete on available RAM (as with CLARK on the 512 GB workstation for AFS31RS90) — record it as '-' rather than a partial peak

## Inputs

- Reference genome collections used to build the AFS20, AFS31, AFS20RS90, and AFS31RS90 databases (FASTA)
- KAL_D paired-end read set, 26,114K read pairs (FASTA/FASTQ)
- Built classifier databases from AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken

## Outputs

- Peak resident set size (GB) per build run for each tool/database combination
- Peak resident set size (GB) per query run for each tool/database combination
- Aggregated TSV table mirroring Table 7 with 16 rows (4 tools x 4 databases) and 5 metrics: build time, peak build memory, query time, query speed (MR/m), query memory

## How to apply

Wrap each build and query invocation in a resource monitor such as /usr/bin/time -v and record the 'Maximum resident set size' as the peak memory for that run. Apply this during both database construction (AFS-MetaCache build mode with the -o remove-overpopulated-features option; CLARK, Kraken2, and Kraken2+Bracken under default construction settings) and classification of the KAL_D paired-end reads (26,114K read pairs) in paired-read mode (-r) with threshold query mode, running MetaCacheSpark with 16 executors and 4 threads per executor (--num-executors 16 --executor-cores 4). The rationale is that peak RAM, alongside wall-clock time, is the decisive scalability metric: in the paper's AFS31RS90 benchmark, AFS-MetaCache peaked at 135 GB while CLARK exceeded the 512 GB workstation memory (reported as '-'), Kraken2 reached 134 GB, and Kraken2+Bracken reached 329 GB. Convert query wall-clock times to speed in millions of reads per minute (MR/m) so the paired time/memory figures form the 5 metrics per tool-by-database row in the aggregated Table 7-style TSV.

## Related tools

- **AFS-MetaCache** (C++ workstation classifier whose build/query runs are profiled for peak memory; fastest and lowest-memory constructor (3h 10m, 135 GB on AFS31RS90)) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark-based classifier executed with 16 executors and 4 threads per executor; its build and query memory consumption is measured alongside runtime) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Comparison classifier whose AFS31RS90 build exceeded the 512 GB workstation memory, recorded as '-' in the benchmark table)
- **Kraken2** (Comparison classifier; peak memory recorded as 134 GB for the AFS31RS90 build)
- **Kraken2+Bracken** (Comparison classifier with abundance estimation; peak build memory recorded as 329 GB for AFS31RS90)

## Examples

```
/usr/bin/time -v ./metacache build afs31rs90.db -o genomes/afs31rs90/ 2>&1 | tee build_afs31rs90.log
```

## Evaluation signals

- The aggregated table contains all 4 tools x 4 databases rows with all 5 metric fields populated (or explicitly '-' when memory was exceeded)
- AFS-MetaCache/MetaCacheSpark show the fastest and lowest-memory database construction, consistent with Table 7 relationships
- Each recorded peak memory corresponds to a 'Maximum resident set size' value from a resource monitor such as /usr/bin/time -v for a completed run
- Query times convert to speeds in MR/m that are internally consistent with the reported wall-clock times for the 26,114K paired reads
- Runs whose memory exceeded the workstation limit (e.g. CLARK on AFS31RS90 with 512 GB RAM) are marked '-' rather than given a fabricated value

## Limitations

- Peak-memory measurement requires RAM exceeding tool needs; CLARK exceeded 512 GB on AFS31RS90 and could not be reported
- The article provides no hardware specifications for the big data cluster used for Spark benchmarks, limiting reproducibility of cluster-side memory measurements
- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), so memory figures may vary across versions

## Evidence

- [results] For the AFS31RS90 database, AFS-MetaCache built fastest (3h 10m, 135 GB peak); CLARK exceeded the 512 GB workstation memory (marked '-'): [evidence span withheld — anchor status: not_found]
- [discussion] AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times: "AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times"
- [methods] Execute MetaCacheSpark for classification with 16 executors and 4 threads per executor.: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [abstract] the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements: "the usage of an efficient database partitioning scheme allows for the processing of massive collections of reference genomes with reduced memory requirements"
- [readme] In order to be able to build a database based on NCBI RefSeq with default settings your system should have at least 128GB of RAM: [evidence span withheld — anchor status: external_doc]
