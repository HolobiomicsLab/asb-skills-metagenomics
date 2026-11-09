---
name: hadoop-mapreduce-job-benchmarking
description: Use when you must verify that a MapReduce-based analysis tool's runtime
  scales acceptably (linearly) with input volume, or when you need a per-metric/per-mode
  runtime breakdown to identify the most expensive computation.
when_to_use_negative:
- Input data are small (single-run, GB-scale or less) where wall-clock timing without
  a scaling sweep suffices and Hadoop overhead dominates.
- The goal is to compare clustering accuracy or distance-metric quality rather than
  runtime/scaling behavior.
- The tool under test is not a Hadoop MapReduce job, since the -s/-m/-w option matrix
  and MapReduce scaling semantics will not apply.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: 'Tool under benchmark: its `core` scoring step is run over synthetic datasets
    of increasing size to measure runtime scaling.'
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Execution framework whose MapReduce run modes (MAP/REDUCE via -m) are exercised
    and timed during the benchmark.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:8621da8d596cc1f8c895d5d4097c304f6998a1751482f0ccb427e3b9b0aff069
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# hadoop-mapreduce-job-benchmarking

## Summary

Benchmark the runtime scaling of a Hadoop MapReduce tool (Libra) across synthetically generated metagenome datasets of increasing input volume (10, 20, 30, and 40 samples of 9.3 GB each, 93–372 GB total), tabulating elapsed times by dataset size, scoring algorithm (-s), and run mode (-m) to verify approximately linear scaling despite quadratic growth in all-vs-all sample pairs.

## When to use

Use when you must verify that a MapReduce-based analysis tool's runtime scales acceptably (linearly) with input volume, or when you need a per-metric/per-mode runtime breakdown to identify the most expensive computation.

## When NOT to use

- Input data are small (single-run, GB-scale or less) where wall-clock timing without a scaling sweep suffices and Hadoop overhead dominates.
- The goal is to compare clustering accuracy or distance-metric quality rather than runtime/scaling behavior.
- The tool under test is not a Hadoop MapReduce job, since the -s/-m/-w option matrix and MapReduce scaling semantics will not apply.

## Inputs

- Synthetic benchmark metagenome FASTA/FASTQ datasets sized at 9.3 GB per sample (groups of 10, 20, 30, and 40 samples; 93–372 GB total)
- Libra k-mer index directories produced by `preprocess` (e.g. -k 20) for each benchmark group
- Hadoop cluster environment

## Outputs

- libra_runtime_benchmark.csv with elapsed runtimes tabulated by dataset size, scoring algorithm (COSINESIMILARITY/BRAYCURTIS/JENSENSHANNON), and run mode (MAP/REDUCE)
- Per-run Libra score directories (via -o /score_dir) for each benchmark configuration
- Computed share of total runtime spent on the most expensive metric (Jensen-Shannon distance, ~12.64% on the 40-sample dataset)

## How to apply

Generate synthetic benchmark inputs (here, synthetic metagenomes at 9.3 GB per sample grouped into 10, 20, 30, and 40 samples) so that input volume grows while the analysis type stays constant. Run Libra scoring on each group via `hadoop jar libra-all.jar core`, iterating over all three scoring algorithms available through the -s option (COSINESIMILARITY default, BRAYCURTIS, JENSENSHANNON) with the default LOGARITHM weighting (-w), and exercising both run modes via -m (MAP default and REDUCE), directing each run's results to an output score directory with -o. Record elapsed wall-clock time per run, noting that all-vs-all sample pairs grow quadratically with sample count while Libra's runtime should grow linearly. Tabulate runtimes by dataset size and scoring algorithm, compute the share of runtime attributable to the most expensive computation (Jensen-Shannon distance ≈12.64% of total runtime on the 40-sample dataset), and save measurements to a CSV (libra_runtime_benchmark.csv).

## Related tools

- **Libra** (Tool under benchmark: its `core` scoring step is run over synthetic datasets of increasing size to measure runtime scaling.) — https://github.com/iychoi/libra
- **Hadoop** (Execution framework whose MapReduce run modes (MAP/REDUCE via -m) are exercised and timed during the benchmark.)

## Examples

```
hadoop jar libra-all.jar core -s JENSENSHANNON -m MAP -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Runtime increases approximately linearly with input volume across the 10–40 sample groups, even though the number of all-vs-all sample pairs grows quadratically.
- All three scoring algorithms (-s COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) are timed, not just the default.
- Both Hadoop run modes (-m MAP and -m REDUCE) appear in the benchmark matrix.
- Jensen-Shannon distance computation accounts for approximately 12.64% of total runtime on the 40-sample dataset.
- All measurements are recorded in a single structured CSV (libra_runtime_benchmark.csv) keyed by dataset size, metric, and mode.

## Limitations

- Scaling claim is validated only on synthetic metagenomes of 9.3 GB per sample (10–40 samples); behavior on real heterogeneous datasets such as Tara Ocean Viromes must be assessed separately.
- Number of sample pairs grows quadratically with sample count, so runtime may deviate from linearity at much larger scale.
- Scoring runs are sensitive to option choices (e.g. Jensen-Shannon is the slowest metric and REDUCE mode differs from the MAP default), so timings are only comparable within a fixed -s/-m/-w configuration.

## Evidence

- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] README scoring command: [evidence span withheld — anchor status: external_doc]
- [readme] README run mode options: "- m : run mode. MAP (default) | REDUCE"
- [abstract] Hadoop scaling finding: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [intro] Scalability objective: "we show that Libra can scale to massive global-scale datasets by examining viral diversity in 43 Tara Ocean Viromes (TOVs)"
