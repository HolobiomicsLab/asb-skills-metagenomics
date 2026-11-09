---
name: metagenome-scalability-runtime-profiling
description: Use when you must verify that Libra's all-vs-all k-mer scoring scales
  linearly with input volume, or when you need per-algorithm (COSINESIMILARITY, BRAYCURTIS,
  JENSENSHANNON) and per-run-mode (MAP vs REDUCE) runtime figures on Hadoop.
when_to_use_negative:
- The goal is clustering accuracy or distance-metric validation on real metagenomes
  (e.g. HMP, CAMI, Tara Ocean Viromes) rather than runtime scaling measurement
- The input data are unindexed raw reads and the question concerns preprocess (k-mer
  indexing) performance rather than the scoring/all-vs-all comparison stage
- The datasets are too small to stress Hadoop (the benchmark design targets 93–372
  GB totals); single-sample or desktop-scale runs will not exhibit the scaling behavior
  being tested
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Libra
  role: Runs the all-vs-all k-mer scoring whose runtime is benchmarked; the -s flag
    selects COSINESIMILARITY, BRAYCURTIS, or JENSENSHANNON and -o directs scores to
    the output directory
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Execution framework whose MapReduce run modes (MAP default, REDUCE via -m)
    are exercised and timed during the scalability benchmark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:921163d9165cccc3161eb53155762a8b00cd553b8db0a3ca55aee4e77e4da224
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenome-scalability-runtime-profiling

## Summary

Benchmarks Libra's all-vs-all metagenome scoring runtime on synthetic Hadoop-scale datasets (10, 20, 30, and 40 samples at 9.3 GB per sample, 93–372 GB total) to verify linear scaling with input volume and to quantify per-metric runtime shares (e.g. Jensen-Shannon ≈12.64% on the 40-sample set).

## When to use

Use when you must verify that Libra's all-vs-all k-mer scoring scales linearly with input volume, or when you need per-algorithm (COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) and per-run-mode (MAP vs REDUCE) runtime figures on Hadoop.

## When NOT to use

- The goal is clustering accuracy or distance-metric validation on real metagenomes (e.g. HMP, CAMI, Tara Ocean Viromes) rather than runtime scaling measurement
- The input data are unindexed raw reads and the question concerns preprocess (k-mer indexing) performance rather than the scoring/all-vs-all comparison stage
- The datasets are too small to stress Hadoop (the benchmark design targets 93–372 GB totals); single-sample or desktop-scale runs will not exhibit the scaling behavior being tested

## Inputs

- Synthetic benchmark metagenome FASTA/FASTQ datasets at 9.3 GB per sample, grouped into sets of 10, 20, 30, and 40 samples (93–372 GB total)
- Libra k-mer index directories produced by the preprocess step (default NOTUNIQUE k-mer filter)
- Libra pre-built binary libra-all.jar (e.g. v1.2.8, compiled with Java 7) on a Hadoop installation

## Outputs

- Runtime benchmark table (CSV, e.g. libra_runtime_benchmark.csv) tabulating elapsed time by dataset size, scoring algorithm, and run mode
- Scalability assessment of whether runtime scales approximately linearly with input volume across the 10–40 sample groups
- Per-metric runtime shares, e.g. Jensen-Shannon distance fraction of total runtime on the 40-sample dataset (≈12.64%)

## How to apply

Generate synthetic benchmark metagenomes at 9.3 GB per sample, forming four groups of 10, 20, 30, and 40 samples, and index each with the Libra preprocess step (k-mer size 20 in the README example) before scoring. Run Libra scoring on each group with 'hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir', sweeping the -s option across COSINESIMILARITY (default), BRAYCURTIS, and JENSENSHANNON, and the -m option across MAP (default) and REDUCE, recording elapsed wall time per run. Note that while input volume grows linearly (10→40 samples), the number of all-vs-all sample pairs grows quadratically, so approximate linearity of runtime is the scaling signal to check. Tabulate runtimes by dataset size and scoring algorithm in a CSV (e.g. libra_runtime_benchmark.csv) and compute the fraction of total 40-sample runtime spent computing the Jensen-Shannon distance (expected ≈12.64%).

## Related tools

- **Libra** (Runs the all-vs-all k-mer scoring whose runtime is benchmarked; the -s flag selects COSINESIMILARITY, BRAYCURTIS, or JENSENSHANNON and -o directs scores to the output directory) — https://github.com/iychoi/libra
- **Hadoop** (Execution framework whose MapReduce run modes (MAP default, REDUCE via -m) are exercised and timed during the scalability benchmark)

## Examples

```
hadoop jar libra-all.jar core -s JENSENSHANNON -m MAP -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Runtime grows approximately linearly with input volume across the 10, 20, 30, and 40 sample groups, despite the all-vs-all sample-pair count growing quadratically
- Jensen-Shannon distance computation accounts for approximately 12.64% of total runtime on the 40-sample dataset
- The benchmark sweep covers all three scoring algorithms (-s COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) under the default LOGARITHM weighting (-w)
- Both Hadoop run modes were exercised (-m MAP and -m REDUCE) and each run mode × algorithm × dataset-size combination has a recorded elapsed time
- All measurements are persisted in a single table (e.g. libra_runtime_benchmark.csv) with consistent columns for dataset size, algorithm, run mode, and runtime

## Limitations

- Linear runtime scaling is claimed for the scoring stage even though the number of all-vs-all pairs grows quadratically, so the benchmark must be interpreted per input volume, not per pair
- Runtime shares such as the ≈12.64% Jensen-Shannon fraction are specific to the 40-sample (372 GB) synthetic dataset and may differ at other sizes
- Timings depend on the chosen Hadoop run mode (MAP vs REDUCE) and weighting (-w LOGARITHM default), so results are not comparable across mode/weighting changes without re-benchmarking

## Evidence

- [abstract] Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir: [evidence span withheld — anchor status: external_doc]
- [readme] m : run mode. MAP (default) | REDUCE: "m : run mode. MAP (default) | REDUCE"
- [readme] s : min size of group in bytes. 10GB by default. For each file group, a separate index file is created.: "s : min size of group in bytes. 10GB by default. For each file group, a separate index file is created."
