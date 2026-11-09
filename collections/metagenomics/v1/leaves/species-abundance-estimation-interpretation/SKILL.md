---
name: species-abundance-estimation-interpretation
description: Use when FASTQ/FASTA reads from a biological (e.g., food) sample must
  be quantified per species against a large reference genome collection, rather than
  only taxonomically labeled.
when_to_use_negative:
- The task is only per-read taxonomic binning with no quantification goal — plain
  query mode without the -a abundance flag suffices.
- The reference collection is a handful of small genomes where an alignment-based
  pipeline (BWA/Bowtie2 per reference genome) is tractable and desired.
- Input is assembled contigs or a feature/OTU table rather than raw sequencing reads
  requiring classification.
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation of the minhashing classifier used to run abundance
    estimation at species level
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark implementation for cluster-scale classification and abundance
    estimation (query mode, -a species)
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Big Data engine executing MetaCacheSpark across cluster executors with multiple
    threads per executor
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Kraken2+Bracken
  role: Comparison baseline for false-positive rate and quantification accuracy in
    benchmarking abundance estimates
- name: CLARK
  role: Comparison baseline for false-positive rate and quantification accuracy in
    benchmarking abundance estimates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:ebd6ac73879774b9548aaf2e05ac880472f898ad945f50fbcc1365e96d644a7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - CLARK
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# species-abundance-estimation-interpretation

## Summary

Estimate and interpret species-level organism abundance in food metagenomics samples using AFS-MetaCache/MetaCacheSpark's abundance-estimation feature, which classifies reads against a minhashing reference database and aggregates per-target assignments at a selectable taxonomic level (e.g., species).

## When to use

Use when FASTQ/FASTA reads from a biological (e.g., food) sample must be quantified per species against a large reference genome collection, rather than only taxonomically labeled.

## When NOT to use

- The task is only per-read taxonomic binning with no quantification goal — plain query mode without the -a abundance flag suffices.
- The reference collection is a handful of small genomes where an alignment-based pipeline (BWA/Bowtie2 per reference genome) is tractable and desired.
- Input is assembled contigs or a feature/OTU table rather than raw sequencing reads requiring classification.

## Inputs

- Paired-end shotgun sequencing reads (FASTQ/FASTA, e.g., KLyo_C paired reads) via the -r paired-reads option
- Reference genome collection database built by MetaCacheSpark (stored in HDFS) or AFS-MetaCache (workstation)
- Taxonomic mapping at species level for abundance aggregation

## Outputs

- Classification output file with per-read taxonomic assignments
- Species-level abundance estimates (per-species read counts/proportions)
- Optional coverage-filter report (e.g., coverage_filter_report.csv) of retained vs. removed candidate genomes

## How to apply

Build (or reuse) a reference database with MetaCacheSpark/AFS-MetaCache, optionally using the -o remove-overpopulated-features option during build to prune overrepresented features. Run query mode with the abundance flag set to the desired taxonomic level (e.g., -a species) so that reads are classified by minhashing k-mer window hits and counts are aggregated per species; enable -r for paired reads. Optionally apply the sequence-coverage filter, which deletes the lowest-coverage quantile (default 10%) of candidate genomes whose covered windows are sparse/uneven before final assignment. Interpret the resulting per-species counts as the sample's composition, benchmarking false-positive rate and quantification accuracy against tools like Kraken2+Bracken and CLARK.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation of the minhashing classifier used to run abundance estimation at species level) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark implementation for cluster-scale classification and abundance estimation (query mode, -a species)) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Big Data engine executing MetaCacheSpark across cluster executors with multiple threads per executor) — https://github.com/jmabuin/MetaCacheSpark
- **Kraken2+Bracken** (Comparison baseline for false-positive rate and quantification accuracy in benchmarking abundance estimates)
- **CLARK** (Comparison baseline for false-positive rate and quantification accuracy in benchmarking abundance estimates)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -a species -r -q threshold KLyo_C_reads_1.fa KLyo_C_reads_2.fa -out results.txt
```

## Evaluation signals

- The classification output reflects quantification superior in false-positive rate and quantification accuracy to Kraken2, CLARK, and Kraken2+Bracken benchmarks.
- Abundance output is aggregated at the requested taxonomic level (e.g., -a species produces per-species counts, not genus-level or read-only output).
- Coverage-filter check: false-positive candidates (sparse/uneven window coverage, e.g., Actinoalloteichus) are removed, while evenly covered genomes (e.g., Brochothrix thermosphacta) are retained.
- Database built with -o remove-overpopulated-features shows reduced feature overrepresentation without losing classification capability.
- Query mode default candidate selection is 'threshold' unless explicitly overridden with -q (precise/threshold/fast/very_fast).

## Limitations

- Database memory footprint grows with the reference collection; very large databases need database partitioning or the Spark version (RefSeq-scale builds want ≥128 GB RAM on a workstation).
- Coverage filtering assumes reads are evenly distributed across genuinely present genomes; genomes with naturally uneven, sparse coverage risk being wrongly deleted.
- Only k-mer sizes up to 16 and specific target/window id widths are supported by default; database and query binaries must use the same compilation variant, and cross-platform/endianness issues can occur.
- The paper does not report software versions, raw data availability, cluster hardware specs, or reference genome identifiers for the benchmarks, limiting exact reproducibility.

## Evidence

- [intro] abundance estimation at selectable taxonomic level: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [abstract] comparison to established tools: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [readme] abundance_per option: "-a,--abundance_per <arg>             [QUERY] Indicates if use the
                                          abundance estimation feature and at
                                          which level."
- [readme] paired reads flag: " -r,--paired_reads                    [QUERY] Use paired reads in the"
- [methods] remove-overpopulated-features option: "* **-o**: Uses the *remove-overpopulated-features*."
- [readme] default query mode: "-q,--query_mode <arg>                [QUERY] Mode selected for query.
                                          Available options are: precise,
                                          threshold,"
