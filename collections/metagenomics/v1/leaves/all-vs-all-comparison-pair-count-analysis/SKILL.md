---
name: all-vs-all-comparison-pair-count-analysis
description: Use when you need to characterize how an all-vs-all metagenome comparison
  tool's runtime scales with input volume, or when validating that Libra's Hadoop
  MapReduce scoring remains near-linear as sample counts grow despite quadratic pair
  growth.
when_to_use_negative:
- You want to evaluate clustering accuracy or distance-metric sensitivity — use dedicated
  validation datasets (HMP, CAMI toy dataset) rather than synthetic scaling data,
  which only measures runtime.
- The input dataset is far smaller than the benchmark range (e.g. a handful of small
  samples), where runtime scaling effects will be dominated by Hadoop startup overhead
  rather than pair-count growth.
- You need presence/absence or sketch-based comparisons (e.g. Mash/Sourmash-style
  MinHash sketches) — this skill specifically benchmarks Libra's full-read k-mer abundance
  scoring, not data-reduced methods.
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Libra
  role: Performs the all-vs-all k-mer comparison and scoring being benchmarked; invoked
    via 'hadoop jar libra-all.jar core' with -s (scoring algorithm), -m (run mode),
    -w (weighting), and -o (output score directory) options.
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Execution framework providing the scalable MapReduce architecture (MAP/REDUCE
    run modes) over which Libra's runtime is measured.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:f44dc83c0e56f1f33fca565e1b24bc5b3a9ff8bef7d6844f68f49104bf9971f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# all-vs-all-comparison-pair-count-analysis

## Summary

Benchmark and analyze the all-vs-all metagenome comparison workload of Libra by running its k-mer scoring on datasets of increasing size (10–40 synthetic samples) and tabulating runtime against the quadratically growing number of sample pairs, to verify near-linear scaling.

## When to use

Use when you need to characterize how an all-vs-all metagenome comparison tool's runtime scales with input volume, or when validating that Libra's Hadoop MapReduce scoring remains near-linear as sample counts grow despite quadratic pair growth.

## When NOT to use

- You want to evaluate clustering accuracy or distance-metric sensitivity — use dedicated validation datasets (HMP, CAMI toy dataset) rather than synthetic scaling data, which only measures runtime.
- The input dataset is far smaller than the benchmark range (e.g. a handful of small samples), where runtime scaling effects will be dominated by Hadoop startup overhead rather than pair-count growth.
- You need presence/absence or sketch-based comparisons (e.g. Mash/Sourmash-style MinHash sketches) — this skill specifically benchmarks Libra's full-read k-mer abundance scoring, not data-reduced methods.

## Inputs

- Synthetic benchmark metagenome FASTA/FASTQ datasets sized at 9.3 GB per sample, grouped into sets of 10, 20, 30, and 40 samples (93–372 GB total)
- Libra index directories produced by the preprocess step (k-mer indices)
- Hadoop cluster with libra-all.jar available

## Outputs

- Per-run elapsed time measurements for each dataset size, scoring algorithm (-s COSINESIMILARITY | BRAYCURTIS | JENSENSHANNON), and Hadoop run mode (-m MAP | REDUCE)
- Runtime scaling table by dataset size and scoring algorithm (e.g. libra_runtime_benchmark.csv)
- Jensen-Shannon distance runtime share of total runtime on the 40-sample dataset (~12.64% expected)
- Libra all-vs-all score matrices written to the score output directory

## How to apply

Generate synthetic benchmark metagenome datasets at 9.3 GB per sample, building groups of 10, 20, 30, and 40 samples (93–372 GB total), and preprocess them into index directories with Libra's preprocess step (e.g. k-mer size 20, NOTUNIQUE filter). Run Libra scoring on each group with the -o option directing scores to an output directory, repeating each run under all three scoring algorithms via -s (COSINESIMILARITY default, BRAYCURTIS, JENSENSHANNON) with the default LOGARITHM weighting (-w), and exercising both run modes via -m (MAP default, REDUCE). Record elapsed time for each run while noting that the number of all-vs-all sample pairs grows quadratically with sample count (10 samples = 45 pairs, 40 samples = 780 pairs). Tabulate runtimes by dataset size and by scoring algorithm, and compute the share of total runtime spent computing the Jensen-Shannon distance on the 40-sample dataset. Save all measurements (e.g. to libra_runtime_benchmark.csv) and accept the benchmark if runtime scales approximately linearly with input volume and Jensen-Shannon accounts for ~12.64% of total runtime on the 40-sample dataset.

## Related tools

- **Libra** (Performs the all-vs-all k-mer comparison and scoring being benchmarked; invoked via 'hadoop jar libra-all.jar core' with -s (scoring algorithm), -m (run mode), -w (weighting), and -o (output score directory) options.) — https://github.com/iychoi/libra
- **Hadoop** (Execution framework providing the scalable MapReduce architecture (MAP/REDUCE run modes) over which Libra's runtime is measured.)

## Examples

```
hadoop jar libra-all.jar core -s JENSENSHANNON -m MAP -w LOGARITHM -o /score_dir /index_dir_40samples
```

## Evaluation signals

- Runtime across the 10, 20, 30, and 40-sample groups increases approximately linearly with input volume, despite the number of all-vs-all sample pairs growing quadratically.
- On the 40-sample dataset, the Jensen-Shannon distance computation accounts for approximately 12.64% of total runtime.
- All three scoring algorithms (-s COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) complete successfully in both run modes (-m MAP and REDUCE) for every dataset size.
- Measured runtimes reproduce the paper's scaling figure (Figs. 6-7) when tabulated by dataset size and scoring algorithm.
- All measurements are consistently captured in a structured table such as libra_runtime_benchmark.csv.

## Limitations

- The number of all-vs-all sample pairs grows quadratically with sample count, so naive implementations will not scale linearly; the benchmark tests whether Libra's Hadoop architecture mitigates this.
- Runtime conclusions are specific to the synthetic 9.3 GB-per-sample metagenome inputs and the benchmarked size range (10–40 samples, 93–372 GB); results may differ for other data types or sizes.
- The Hadoop framework introduces constant overhead (job setup, reducers) that can dominate runtime on small datasets, masking scaling effects.

## Evidence

- [results] Linear scaling of runtime with input volume: [evidence span withheld — anchor status: not_found]
- [abstract] Libra uses all reads in scalable Hadoop architecture: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset"
- [readme] Scoring options from README: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON - m : run mode. MAP (default) | REDUCE"
- [readme] Scoring invocation example: [evidence span withheld — anchor status: external_doc]
