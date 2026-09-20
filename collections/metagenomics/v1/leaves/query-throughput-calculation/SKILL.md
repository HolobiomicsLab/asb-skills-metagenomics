---
name: query-throughput-calculation
description: Use when benchmarking read classifiers and you must normalize query wall-clock
  times into MR/m so that tools run on different databases or hardware can be compared
  fairly.
when_to_use_negative:
- When the goal is classification accuracy (false-positive rate or quantification
  accuracy) rather than runtime performance — throughput says nothing about correctness
  of assignments.
- When query times were measured with inconsistent thread/executor settings across
  tools, since MR/m values would not be comparable.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier whose query wall-clock time is converted to MR/m
    throughput
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark classifier run with 16 executors/4 threads whose query time is converted
    to MR/m
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Baseline classifier whose query time is converted to MR/m for comparison
- name: Kraken2
  role: Baseline classifier whose query time is converted to MR/m for comparison
- name: Kraken2+Bracken
  role: Baseline classifier (with abundance estimation) whose query time is converted
    to MR/m
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:005570cc32a2fc237176118f6bc73d1fbf165e874a2c44bcfec9bbb40bffda77
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# query-throughput-calculation

## Summary

Converts metagenomic classifier query wall-clock times into throughput expressed in millions of reads per minute (MR/m), as done in the AFS benchmark (Table 7) comparing AFS-MetaCache, CLARK, Kraken2, and Kraken2+Bracken across AFS20/AFS31/AFS20RS90/AFS31RS90 databases.

## When to use

Use when benchmarking read classifiers and you must normalize query wall-clock times into MR/m so that tools run on different databases or hardware can be compared fairly.

## When NOT to use

- When the goal is classification accuracy (false-positive rate or quantification accuracy) rather than runtime performance — throughput says nothing about correctness of assignments.
- When query times were measured with inconsistent thread/executor settings across tools, since MR/m values would not be comparable.

## Inputs

- Query wall-clock time measurements (seconds or h/m/s) for each tool/database combination
- Total read count of the query set (KAL_D: 26,114K paired-end read pairs)
- Classifier/database combination identifiers (AFS-MetaCache, MetaCacheSpark, CLARK, Kraken2, Kraken2+Bracken x AFS20, AFS31, AFS20RS90, AFS31RS90)

## Outputs

- Query throughput per tool/database combination in MR/m (millions of reads per minute)
- A TSV table mirroring Table 7 with 5 metrics (build time, peak build memory, query time, query speed, query memory) for 16 tool-by-database rows

## How to apply

Classify the KAL_D paired-end read set (26,114K read pairs) against each reference database with each classifier, enabling paired-read mode (-r) and threshold query mode, and record query wall-clock time (e.g. via /usr/bin/time -v). Divide the total number of processed reads (both mates of each pair, per the paper's convention) by the elapsed minutes to obtain speed in MR/m. For MetaCacheSpark, execute with 16 executors and 4 threads per executor (--num-executors 16 --executor-cores 4) before timing. Aggregate the resulting throughput values with build time, peak build memory, query time, and query memory into the 16 tool-by-database rows mirroring Table 7, and validate that relative ordering (AFS-MetaCache/MetaCacheSpark fastest) matches the paper.

## Related tools

- **AFS-MetaCache** (C++ workstation classifier whose query wall-clock time is converted to MR/m throughput) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark classifier run with 16 executors/4 threads whose query time is converted to MR/m) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Baseline classifier whose query time is converted to MR/m for comparison)
- **Kraken2** (Baseline classifier whose query time is converted to MR/m for comparison)
- **Kraken2+Bracken** (Baseline classifier (with abundance estimation) whose query time is converted to MR/m)

## Examples

```
/usr/bin/time -v metacache query AFS31RS90 KAL_D_1.fa KAL_D_2.fa -pairseq -r -query_mode threshold -out results.txt; python -c "print(26114*2/(ELAPSED_MIN))"  # MR/m
```

## Evaluation signals

- Each of the 16 tool-by-database rows has a populated query-speed field in MR/m with plausible magnitude (order of tens to hundreds of MR/m).
- MR/m values are internally consistent with the recorded query times: reads_total / (time_seconds/60) reproduces the reported value.
- Relative trend matches the paper: AFS-MetaCache/MetaCacheSpark show highest query throughput compared to CLARK, Kraken2, and Kraken2+Bracken.
- Entries where a tool exceeded the 512 GB workstation memory (e.g. CLARK on AFS31RS90) are marked '-' rather than given a fabricated throughput.
- Throughput is computed on the same read-count basis (paired KAL_D reads) for every tool so cross-tool comparison is valid.

## Limitations

- Throughput depends strongly on database size and hardware; only relative comparisons under identical settings are meaningful.
- CLARK exceeded the 512 GB workstation memory on AFS31RS90, so no query throughput could be recorded for that combination.
- The article does not report software versions for the benchmarked tools, so throughput values may not be reproducible exactly with newer releases.

## Evidence

- [other] Record query wall-clock time and peak query memory for each tool/database combination, converting query time to speed in millions of reads per minute (MR/m).: [evidence span withheld — anchor status: not_found]
- [other] Classify the KAL_D paired-end reads (26,114K read pairs) against each of the four databases: [evidence span withheld — anchor status: not_found]
- [methods] Execute MetaCacheSpark for classification with 16 executors and 4 threads per executor.: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [methods] threshold query mode per the paper's configuration: "Available options are: precise, threshold, fast, very_fast. Default: threshold"
