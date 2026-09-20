---
name: abundance-quantification-accuracy-assessment
description: Use when benchmarking taxonomic classifiers (e.g. AFS-MetaCache, MetaCacheSpark,
  CLARK, Kraken2, Kraken2+Bracken) on samples with known ingredient compositions to
  quantify false-positive rates and abundance-estimation error.
when_to_use_negative:
- Samples have unknown composition — no ground truth exists to compute Σ FP or Σ Dev,
  so accuracy assessment is meaningless
- The goal is database construction or read-level classification speed benchmarking
  rather than quantification accuracy
- Input is raw unassembled reads without a corresponding reference database (e.g.
  AFS20) — classification cannot proceed
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: AFS-MetaCache
  role: C++ workstation classifier under evaluation; reported lowest Σ FP and Σ Dev
    on calibrator samples
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark-based classifier run in query mode (-m query, default 'threshold' query
    mode) on the calibrator datasets and used to build the AFS20 database
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: CLARK
  role: Established baseline classifier (default parameters) benchmarked for Σ FP
    and Σ Dev
- name: Kraken2
  role: Established baseline classifier (default parameters) benchmarked for Σ FP
    and Σ Dev
- name: Kraken2+Bracken
  role: Established baseline classifier with abundance re-estimation (default parameters)
    benchmarked for Σ FP and Σ Dev
- name: Apache Spark
  role: Compute engine for running MetaCacheSpark build/query jobs on YARN/HDFS (16
    executors, 4 threads per executor for query)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:b804d3bce83f8161d0b523c458e080b5b9f16c0988f8904fd8e5ec14fdc6dc6d
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# abundance-quantification-accuracy-assessment

## Summary

Evaluates metagenomic classifier quantification accuracy by classifying calibrator food samples of known composition (Kal A-E and KLyo A-D calibrator sausage read datasets) and computing per-sample sums of false positives (Σ FP) and sums of absolute deviations (Σ Dev) between predicted and true ingredient abundances.

## When to use

Use when benchmarking taxonomic classifiers (e.g. AFS-MetaCache, MetaCacheSpark, CLARK, Kraken2, Kraken2+Bracken) on samples with known ingredient compositions to quantify false-positive rates and abundance-estimation error.

## When NOT to use

- Samples have unknown composition — no ground truth exists to compute Σ FP or Σ Dev, so accuracy assessment is meaningless
- The goal is database construction or read-level classification speed benchmarking rather than quantification accuracy
- Input is raw unassembled reads without a corresponding reference database (e.g. AFS20) — classification cannot proceed

## Inputs

- Kal A-E calibrator sausage read datasets (HDFS input sequences)
- KLyo A-D calibrator sausage read datasets (HDFS input sequences)
- Known calibrator ingredient compositions (ground truth)
- AFS20 reference database built with MetaCacheSpark (build mode, -o remove-overpopulated-features, -t HDFS taxonomy path)
- Classifier outputs from AFS-MetaCache, MetaCacheSpark, CLARK, Kraken2, and Kraken2+Bracken

## Outputs

- Per-sample Σ FP (sum of false positives) for each classifier and calibrator sample
- Per-sample Σ Dev (sum of absolute deviations between predicted and true ingredient abundances) for each classifier
- Average Σ FP and Σ Dev across the nine calibrator samples per classifier
- classifier_comparison.tsv reproducing Tables 5 and 6

## How to apply

Classify the Kal A-E and KLyo A-D calibrator sausage read datasets (stored as sequences on HDFS) with each classifier under default parameters — AFS-MetaCache and MetaCacheSpark via the Spark query mode (-m query) with the default 'threshold' query mode against the AFS20 reference database. From each classifier's output, compute per-sample Σ FP against the known calibrator compositions and Σ Dev between predicted and true ingredient abundances (the abundance estimation level is set with the -a/--abundance_per option). Average Σ FP and Σ Dev across the nine calibrator samples per classifier and tabulate per-sample and average values (reproducing Tables 5 and 6). The rationale is that calibrators of known composition provide ground truth for both detection (FP) and quantification (Dev) accuracy; the paper reports AFS-MetaCache achieves the lowest Σ FP (0.67% KLyo / 1.12% Kal on average) versus CLARK (1.34%/3.59%), Kraken2 (1.86%/3.87%) and Kraken2+Bracken (2.14%/4.41%).

## Related tools

- **AFS-MetaCache** (C++ workstation classifier under evaluation; reported lowest Σ FP and Σ Dev on calibrator samples) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark-based classifier run in query mode (-m query, default 'threshold' query mode) on the calibrator datasets and used to build the AFS20 database) — https://github.com/jmabuin/MetaCacheSpark
- **CLARK** (Established baseline classifier (default parameters) benchmarked for Σ FP and Σ Dev)
- **Kraken2** (Established baseline classifier (default parameters) benchmarked for Σ FP and Σ Dev)
- **Kraken2+Bracken** (Established baseline classifier with abundance re-estimation (default parameters) benchmarked for Σ FP and Σ Dev)
- **Apache Spark** (Compute engine for running MetaCacheSpark build/query jobs on YARN/HDFS (16 executors, 4 threads per executor for query))

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark -m query -q threshold -a species -n 4 MetaCacheSpark-0.4.0.jar AFS20 kal_reads
```

## Evaluation signals

- Σ FP and Σ Dev values are computed for all five classifiers across all nine Kal and KLyo calibrator samples
- Average Σ FP per classifier falls in the reported ranges (e.g. AFS-MetaCache ≈0.67% KLyo / 1.12% Kal; Kraken2+Bracken ≈2.14%/4.41%)
- Predicted ingredient abundances can be directly compared against the known calibrator compositions for every sample
- Result table written to classifier_comparison.tsv reproduces the structure of Tables 5 and 6 (per-sample plus average values per classifier)
- AFS-MetaCache and MetaCacheSpark show lower Σ FP and Σ Dev than CLARK, Kraken2, and Kraken2+Bracken, matching the paper's reported finding

## Limitations

- Comparisons rely on default parameters for all tools; no software version numbers are given for the benchmarked tools (Kraken2, CLARK, Bracken, Spark)
- Raw sequencing data or reference sample datasets used for the admixture benchmarks have no stated availability, limiting reproducibility
- Hardware specifications of the big data cluster used for the benchmarks are not provided

## Evidence

- [abstract] In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy.: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [methods] query mode threshold default: "* *threshold*: A threshold value is calculated according different parameters."
- [methods] query execution with 16 executors: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [methods] remove overpopulated features option: "**-o**: Uses the *remove-overpopulated-features*."
- [intro] abundance estimation at selectable level: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [readme] query mode options readme: "-q,--query_mode <arg>                [QUERY] Mode selected for query."
