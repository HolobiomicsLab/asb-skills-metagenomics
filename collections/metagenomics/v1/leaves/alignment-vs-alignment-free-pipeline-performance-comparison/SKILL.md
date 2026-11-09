---
name: alignment-vs-alignment-free-pipeline-performance-comparison
description: Use when replacing or validating an alignment-per-genome pipeline (BWA/Bowtie2/CUSHAW)
  with an alignment-free classifier and you must quantify speed-up (build time, MR/m
  query speed) and accuracy trade-offs (Σ FP, Σ Dev) on identical inputs.
when_to_use_negative:
- The goal is classification accuracy benchmarking against other metagenomic classifiers
  (CLARK, Kraken2, Kraken2+Bracken) rather than alignment-based vs alignment-free
  pipeline comparison.
- Input data are already classified abundance tables, not raw reads and reference
  genomes.
- Only a single pipeline can be run — the skill requires running both pipelines on
  the same samples and database for a fair side-by-side comparison.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: BWA
  role: Alignment tool of the legacy AFS pipeline, run once per each of the 10 AFS10
    reference genomes to produce the baseline build time and query speed
- name: AFS-MetaCache
  role: Alignment-free minhashing-based classifier whose database build and query
    performance and quantification accuracy (Σ FP, Σ Dev) are benchmarked against
    the BWA pipeline
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark-based variant of MetaCache that can substitute for the workstation
    C++ version on big data clusters
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Execution engine for MetaCacheSpark when running the comparison on a big data
    cluster (e.g., 16 executors, 4 threads per executor for query)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:3549d4f3d90f63ff6accb8c4e812a0a114ee4d6a7d9b17f3a7f998d6a2e670c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - BWA
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# alignment-vs-alignment-free-pipeline-performance-comparison

## Summary

Benchmarks an alignment-based pipeline (BWA per reference genome) against the alignment-free, k-mer/minhash-based AFS-MetaCache on the same reference database and read samples, tabulating database build time, query speed (MR/m), and quantification accuracy (Σ FP, Σ Dev) as reported in Tables 9-10 of the all-food-sequencing study.

## When to use

Use when replacing or validating an alignment-per-genome pipeline (BWA/Bowtie2/CUSHAW) with an alignment-free classifier and you must quantify speed-up (build time, MR/m query speed) and accuracy trade-offs (Σ FP, Σ Dev) on identical inputs.

## When NOT to use

- The goal is classification accuracy benchmarking against other metagenomic classifiers (CLARK, Kraken2, Kraken2+Bracken) rather than alignment-based vs alignment-free pipeline comparison.
- Input data are already classified abundance tables, not raw reads and reference genomes.
- Only a single pipeline can be run — the skill requires running both pipelines on the same samples and database for a fair side-by-side comparison.

## Inputs

- AFS10 reference genome collection (10 food-relevant reference genomes)
- Calibrator sausage read datasets (KAL_D dataset and Kal/KLyos samples) in FASTA/FASTQ
- BWA alignment outputs per reference genome from the legacy AFS pipeline

## Outputs

- Comparison table (e.g., afs_pipeline_comparison_tables_9_10.tsv) with build time, query speed (MR/m), Σ FP, and Σ Dev per pipeline
- Recorded database build times for BWA-based indexing and AFS-MetaCache construction
- Per-sample abundance/quantification estimates at species level from AFS-MetaCache

## How to apply

First run the legacy pipeline: align the KAL_D dataset and Kal/KLyos sausage samples against each of the 10 AFS10 reference genomes using BWA. Then build the AFS-MetaCache database from the same AFS10 reference genome collection using its minhashing-based k-mer index, recording build time (the legacy index took 7 h versus 47 m for AFS-MetaCache). Classify the samples with AFS-MetaCache using abundance estimation at the species level plus the remove-overpopulated-features filtering option (-o), recording query speed in millions of reads per minute (17.1 MR/m vs 0.04 MR/m for BWA). For accuracy, compute the sum of false positives (Σ FP) and sum of deviations (Σ Dev) for both pipelines on the same samples so errors are directly attributable to the method rather than the data. Tabulate build time, query speed, Σ FP, and Σ Dev side by side and export as a TSV (e.g., afs_pipeline_comparison_tables_9_10.tsv) matching the published Tables 9 and 10.

## Related tools

- **BWA** (Alignment tool of the legacy AFS pipeline, run once per each of the 10 AFS10 reference genomes to produce the baseline build time and query speed)
- **AFS-MetaCache** (Alignment-free minhashing-based classifier whose database build and query performance and quantification accuracy (Σ FP, Σ Dev) are benchmarked against the BWA pipeline) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark-based variant of MetaCache that can substitute for the workstation C++ version on big data clusters) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Execution engine for MetaCacheSpark when running the comparison on a big data cluster (e.g., 16 executors, 4 threads per executor for query))

## Examples

```
metacache query afs10 KAL_D_reads.fa -species -o -out results.txt
```

## Evaluation signals

- AFS10 database build time for AFS-MetaCache should be far below the ~7 h BWA-based index construction (published: 47 m).
- Query speed on KAL_D should improve by more than two orders of magnitude (published: 17.1 MR/m for AFS-MetaCache vs 0.04 MR/m for BWA).
- Σ FP and Σ Dev computed for both pipelines on the same Kal/KLyos samples should match the values in published Tables 9-10.
- Output TSV columns must contain all four metrics (build time, query speed, Σ FP, Σ Dev) for both pipelines with identical sample sets.
- Order-of-magnitude speed-up values should pass expert review for plausibility given the reference database size.

## Limitations

- The comparison is specific to the AFS10 reference set and calibrator sausage datasets; speed-ups may not transfer to other genome collections or sample types.
- Hardware specifications for the benchmark cluster are not provided in the article, limiting reproducibility of absolute timings.
- No software version numbers are given for the benchmarked tools, so exact version-dependent performance cannot be guaranteed.

## Evidence

- [abstract] alignment-free vs alignment pipeline finding: "It is orders-of-magnitude faster than our previous alignment-based AFS pipeline."
- [intro] legacy pipeline relies on per-genome alignment: "the AFS pipeline relies on applying a read alignment tool (such as BWA [7–9], Bowtie2 [10], or CUSHAW [11]) for each considered reference genome"
- [methods] workflow on Kal and KLyos with abundance and filtering options: "* **-o**: Uses the *remove-overpopulated-features*."
- [methods] MetaCacheSpark execution parameters: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [discussion] superiority in FP rates: "The comparison to the established metagenomics tools Kraken2, CLARK, and Kraken2+Bracken shows that AFS-MetaCache and MetaCacheSpark are superior in terms of false positive (FP) rates."
