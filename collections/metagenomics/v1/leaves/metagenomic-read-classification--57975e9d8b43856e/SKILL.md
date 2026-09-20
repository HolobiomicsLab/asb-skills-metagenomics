---
name: metagenomic-read-classification
description: Use when classifying shotgun FASTQ/FASTA reads from complex biological
  samples (e.g., calibrator sausage mixtures) against large reference genome collections
  to detect ingredient/species composition and estimate abundances with low false-positive
  rates.
when_to_use_negative:
- When a full alignment-based pipeline is required (e.g., SNP-level variant calling)
  — the minhashing approach is a classification/quantification substitute for alignment
  tools like BWA/Bowtie2, not a read aligner
- When the reference database must be shared between MetaCache variants — a database
  can only be queried with the same compiled variant (data type sizes, endianness)
  that built it
- When the target organisms lack reference genomes in the database, since classification
  depends entirely on indexed reference sequences
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation used for minhashing-based read classification
    and abundance estimation against partitioned reference databases
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark implementation used to build (-m build) and query (-m query)
    the AFS reference databases on a YARN/HDFS big data cluster
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Established metagenomic classifier used as benchmark baseline (default parameters)
    for Σ FP and Σ Dev comparison
- name: Kraken2
  role: Established k-mer classifier used as benchmark baseline (default parameters)
    for Σ FP and Σ Dev comparison
- name: Kraken2+Bracken
  role: Kraken2 with abundance re-estimation used as benchmark baseline (default parameters)
    for Σ FP and Σ Dev comparison
- name: Apache Spark
  role: Execution engine running MetaCacheSpark with 16 executors (4 threads per executor
    in query mode) on the big data cluster
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:1b7034eae07d14c2045dfd1e52491c3431f3b54b1dc7bb89d72bdf076d517538
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Hadoop
  - Apache Spark
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# metagenomic-read-classification

## Summary

Alignment-free, minhashing-based taxonomic classification and abundance estimation of metagenomic sequencing reads against large multi-kingdom reference databases, as implemented in AFS-MetaCache (C++/workstation) and MetaCacheSpark (Apache Spark cluster) for all-food-sequencing (AFS) analyses.

## When to use

Use when classifying shotgun FASTQ/FASTA reads from complex biological samples (e.g., calibrator sausage mixtures) against large reference genome collections to detect ingredient/species composition and estimate abundances with low false-positive rates.

## When NOT to use

- When a full alignment-based pipeline is required (e.g., SNP-level variant calling) — the minhashing approach is a classification/quantification substitute for alignment tools like BWA/Bowtie2, not a read aligner
- When the reference database must be shared between MetaCache variants — a database can only be queried with the same compiled variant (data type sizes, endianness) that built it
- When the target organisms lack reference genomes in the database, since classification depends entirely on indexed reference sequences

## Inputs

- Shotgun sequencing reads (FASTA/FASTQ, single or paired) from food or other complex biological samples (e.g., Kal A-E and KLyo A-D calibrator sausage datasets on HDFS)
- Reference genome collections spanning multiple kingdoms (e.g., the AFS20 or AFS31RS90 reference databases)
- NCBI taxonomy files (path passed via -t)

## Outputs

- Per-read taxonomic assignments (classification output file)
- Species-level abundance estimates (via -a abundance_per option)
- Benchmark tables of Σ FP and Σ Dev per classifier and sample (e.g., classifier_comparison.tsv reproducing Tables 5-6)
- Coverage-filter decisions on candidate genomes (e.g., coverage_filter_report.csv)

## How to apply

Build a reference database with MetaCacheSpark in build mode (-m build) on YARN/HDFS, enabling the remove-overpopulated-features option (-o) and passing the HDFS taxonomy path with -t; partitioning lets massive eukaryotic+bacterial genome collections fit in reduced memory. Classify reads in query mode (-m query, default threshold query mode) with 16 executors and 4 threads per executor, optionally enabling paired reads (-r) and abundance estimation at a selectable taxonomic level (e.g., -a species). Compute per-sample sums of false positives (Σ FP) against known sample compositions and sums of absolute deviations (Σ Dev) between predicted and true ingredient abundances to benchmark against CLARK, Kraken2, and Kraken2+Bracken with default parameters. Optionally apply the coverage filter, which deletes the lowest-coverage quantile (e.g., 10%) of candidate genomes whose covered windows are sparse/uneven before fixing final assignments. Correct application is judged when classification and quantification outperform the established tools in Σ FP and Σ Dev, as reported for the Kal A-E and KLyo A-D calibrator datasets against the AFS20 database.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation used for minhashing-based read classification and abundance estimation against partitioned reference databases) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark implementation used to build (-m build) and query (-m query) the AFS reference databases on a YARN/HDFS big data cluster) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Established metagenomic classifier used as benchmark baseline (default parameters) for Σ FP and Σ Dev comparison)
- **Kraken2** (Established k-mer classifier used as benchmark baseline (default parameters) for Σ FP and Σ Dev comparison)
- **Kraken2+Bracken** (Kraken2 with abundance re-estimation used as benchmark baseline (default parameters) for Σ FP and Σ Dev comparison)
- **Apache Spark** (Execution engine running MetaCacheSpark with 16 executors (4 threads per executor in query mode) on the big data cluster)

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -r -a species -o -t /hdfs/taxonomy
```

## Evaluation signals

- Per-sample Σ FP computed against known calibrator compositions averages ~0.67% (KLyo) and ~1.12% (Kal) for AFS-MetaCache, lower than CLARK (1.34%/3.59%), Kraken2 (1.86%/3.87%), and Kraken2+Bracken (2.14%/4.41%)
- Σ Dev between predicted and true ingredient abundances is computed for all five classifiers across all nine calibrator samples (Kal A-E + KLyo A-D), reproducing Tables 5-6
- Coverage filter removes sparse/uneven-coverage candidates (e.g., Actinoalloteichus) while retaining even-coverage genomes (e.g., Brochothrix thermosphacta), matching Figs. 5-6 and Table 11
- Query results use the default 'threshold' query mode unless otherwise specified (options: precise, threshold, fast, very_fast)
- The database build completes with remove-overpopulated-features enabled and the resulting database is stored in HDFS

## Limitations

- No software version numbers were reported for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), complicating exact reproduction
- Hardware specifications of the big data cluster used for benchmarks are not provided
- No specific versions or identifiers of the reference genomes used for database construction are stated
- A database can only be queried with the same MetaCache variant (data type sizes, k-mer length, endianness) that built it
- Full RefSeq-scale database builds need at least 128GB RAM without partitioning, per the README

## Evidence

- [abstract] superiority over established classifiers: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [intro] abundance estimation and coverage filtering: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [methods] query execution parameters: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [methods] query mode options: "* *threshold*: A threshold value is calculated according different parameters."
- [readme] tool description: "MetaCacheSpark performs metagenomics analysis by means of Apache Spark and the metacache minhashing algorithm."
