---
name: false-positive-rate-benchmarking
description: Use when comparing metagenomic classifiers on samples with known ground-truth
  ingredient composition, to quantify false positives and abundance-estimation accuracy
  per tool.
when_to_use_negative:
- Samples have no known ground-truth ingredient composition, so false positives and
  deviations cannot be computed against true abundances
- The goal is raw read classification speed or database build benchmarking rather
  than false-positive rate and quantification accuracy
- Input is a single-genome re-sequencing project rather than multi-ingredient metagenomic
  mixtures spanning kingdoms of life
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: AFS-MetaCache
  role: k-mer/minhashing classifier under benchmark; expected to achieve lowest Σ
    FP and Σ Dev
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark-based classifier and database builder (build/query modes) used in the
    comparison
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: established classifier used as default-parameter baseline for Σ FP comparison
- name: Kraken2
  role: established classifier used as default-parameter baseline for Σ FP comparison
- name: Kraken2+Bracken
  role: established classifier plus abundance re-estimation used as baseline
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:dcc5ab0279eaa4639da842f1faa25e342952ac003f49072d71ff53bd42ac9fca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Hadoop
  - BWA
  - CLARK
  - Kraken2
  - Kraken2+Bracken
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# false-positive-rate-benchmarking

## Summary

Benchmark false-positive rates (Σ FP) and absolute-deviation sums (Σ Dev) of metagenomic classifiers (AFS-MetaCache, MetaCacheSpark, CLARK, Kraken2, Kraken2+Bracken) on calibrator samples of known composition, as done for all-food-sequencing sausage datasets against the AFS20 reference database.

## When to use

Use when comparing metagenomic classifiers on samples with known ground-truth ingredient composition, to quantify false positives and abundance-estimation accuracy per tool.

## When NOT to use

- Samples have no known ground-truth ingredient composition, so false positives and deviations cannot be computed against true abundances
- The goal is raw read classification speed or database build benchmarking rather than false-positive rate and quantification accuracy
- Input is a single-genome re-sequencing project rather than multi-ingredient metagenomic mixtures spanning kingdoms of life

## Inputs

- Calibrator read datasets with known composition (Kal A-E and KLyo A-D calibrator sausage FASTA/FASTQ reads, e.g. in HDFS)
- Reference database (e.g., AFS20 built with MetaCacheSpark) plus taxonomy files
- Classifier outputs (per-read taxonomic assignments and abundance estimates) from AFS-MetaCache, MetaCacheSpark, CLARK, Kraken2, Kraken2+Bracken

## Outputs

- Per-sample Σ FP (sum of false positives) per classifier
- Per-sample Σ Dev (sum of absolute deviations between predicted and true ingredient abundances) per classifier
- Averaged Σ FP and Σ Dev across all calibrator samples per classifier (comparison table, e.g. classifier_comparison.tsv)

## How to apply

Build the reference database (e.g., AFS20 with MetaCacheSpark in build mode via spark-submit, enabling the -o remove-overpopulated-features option and passing the HDFS taxonomy path with -t). Classify each calibrator dataset (Kal A-E, KLyo A-D calibrator sausage reads) with all classifiers using default parameters, and with AFS-MetaCache using abundance estimation at the species level. For each classifier and sample, compute the sum of false positives (Σ FP) against the known calibrator composition and the sum of absolute deviations (Σ Dev) between predicted and true ingredient abundances. Average Σ FP and Σ Dev across the nine calibrator samples per classifier and tabulate per-sample and average values (Tables 5-6 pattern). The method is accepted as applied correctly when AFS-MetaCache/MetaCacheSpark show lower Σ FP than CLARK, Kraken2, and Kraken2+Bracken, reproducing the paper's reported averages (e.g., KLyo 0.67% vs CLARK 1.34%, Kraken2 1.86%, Kraken2+Bracken 2.14%).

## Related tools

- **AFS-MetaCache** (k-mer/minhashing classifier under benchmark; expected to achieve lowest Σ FP and Σ Dev) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark-based classifier and database builder (build/query modes) used in the comparison) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (established classifier used as default-parameter baseline for Σ FP comparison)
- **Kraken2** (established classifier used as default-parameter baseline for Σ FP comparison)
- **Kraken2+Bracken** (established classifier plus abundance re-estimation used as baseline)

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -t hdfs://taxonomy
```

## Evaluation signals

- Σ FP and Σ Dev computed for all five classifiers across all nine Kal and KLyo calibrator samples
- AFS-MetaCache average Σ FP ≈ 0.67% (KLyo) and 1.12% (Kal), matching the published Tables 5-6 values
- Baseline averages match reported values: CLARK 1.34%/3.59%, Kraken2 1.86%/3.87%, Kraken2+Bracken 2.14%/4.41%
- Ranking invariants hold: AFS-MetaCache and MetaCacheSpark ≤ CLARK < Kraken2 < Kraken2+Bracken in false-positive rate
- Per-sample Σ FP values are reproducible against the known calibrator compositions (no missing samples in the table)

## Limitations

- No software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark), limiting exact reproducibility
- No availability statement for the raw sequencing data or reference sample datasets used for the admixture benchmarks
- Hardware specifications of the big data cluster used for benchmarks are not provided

## Evidence

- [results] AFS-MetaCache achieves the lowest false-positive rates across all tested calibrator sausage datasets: [evidence span withheld — anchor status: not_found]
- [abstract] superior to established tools: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [methods] remove-overpopulated-features option: "**-o**: Uses the *remove-overpopulated-features*."
- [readme] default query mode: "Available options are: precise, threshold, fast, very_fast. Default: threshold"
- [readme] abundance estimation level: "-a,--abundance_per <arg>             [QUERY] Indicates if use the
                                          abundance estimation feature and at
                                          which level."
