---
name: distance-metric-runtime-comparison
description: Use when you must reproduce or verify runtime-scaling claims for an all-vs-all
  metagenome comparison tool, or quantify the per-metric runtime share of Libra's
  scoring algorithms across increasing input volumes.
when_to_use_negative:
- The goal is clustering accuracy or distance-metric quality on real microbiomes (e.g.
  HMP 16S/WGS or CAMI toy datasets) rather than runtime scaling — use Libra's validation
  benchmarks instead
- Datasets are far smaller than the benchmark scale, where per-run times are too short
  to distinguish linear from quadratic scaling
- The scoring/k-mer index has not yet been built — this skill times the core scoring
  stage only, not FASTA/FASTQ preprocessing
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: Libra
  role: Runs the timed all-vs-all scoring (core) stage with selectable -s scoring
    algorithms, -w weighting, and -m MAP/REDUCE run modes on the synthetic index
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: MapReduce execution framework on which Libra scoring is launched and elapsed
    times are recorded for each run mode
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:f816cea53db3847f4a97875a7e390a2a1dffec2d5112b19bbaabddea3077b926
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# distance-metric-runtime-comparison

## Summary

Benchmarks how Libra's all-vs-all metagenome comparison runtime scales with input volume and how much runtime each scoring algorithm (COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) consumes, using synthetic 9.3 GB/sample datasets run on Hadoop. This validates the claim that Libra scales linearly despite the quadratically growing number of all-vs-all sample pairs.

## When to use

Use when you must reproduce or verify runtime-scaling claims for an all-vs-all metagenome comparison tool, or quantify the per-metric runtime share of Libra's scoring algorithms across increasing input volumes.

## When NOT to use

- The goal is clustering accuracy or distance-metric quality on real microbiomes (e.g. HMP 16S/WGS or CAMI toy datasets) rather than runtime scaling — use Libra's validation benchmarks instead
- Datasets are far smaller than the benchmark scale, where per-run times are too short to distinguish linear from quadratic scaling
- The scoring/k-mer index has not yet been built — this skill times the core scoring stage only, not FASTA/FASTQ preprocessing

## Inputs

- Synthetic metagenome datasets of 9.3 GB per sample, in four groups of 10, 20, 30, and 40 samples (93–372 GB total), as FASTA/FASTQ
- Libra k-mer index directories produced by the preprocess stage
- Hadoop cluster/environment with the libra-all.jar binary

## Outputs

- libra_runtime_benchmark.csv: runtime measurements tabulated by dataset size, scoring algorithm, and run mode
- Runtime scaling table/figure showing elapsed time versus input volume (10–40 samples)
- Per-metric runtime shares (e.g. Jensen-Shannon distance fraction of total runtime on the 40-sample dataset)

## How to apply

Generate synthetic benchmark metagenome datasets at 9.3 GB per sample and form four groups of 10, 20, 30, and 40 samples (93–372 GB total), so the number of all-vs-all pairs grows quadratically while input volume grows linearly. Preprocess each group into a k-mer index (e.g. hadoop jar libra-all.jar preprocess -k 20), then run Libra scoring with -w LOGARITHM under all three -s scoring algorithms (COSINESIMILARITY default, BRAYCURTIS, JENSENSHANNON) and both -m run modes (MAP default and REDUCE), directing scores to an output directory with -o. Record elapsed wall-clock time for every dataset-size × algorithm × run-mode combination, tabulate runtimes by dataset size and by scoring algorithm, and compute the Jensen-Shannon distance share of total runtime on the 40-sample dataset. Save all measurements to libra_runtime_benchmark.csv and accept the benchmark only if runtime scales approximately linearly with input volume and Jensen-Shannon accounts for roughly 12.64% of total runtime on the 40-sample group.

## Related tools

- **Libra** (Runs the timed all-vs-all scoring (core) stage with selectable -s scoring algorithms, -w weighting, and -m MAP/REDUCE run modes on the synthetic index) — https://github.com/iychoi/libra
- **Hadoop** (MapReduce execution framework on which Libra scoring is launched and elapsed times are recorded for each run mode)

## Examples

```
hadoop jar libra-all.jar core -s JENSENSHANNON -m REDUCE -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Runtime increases approximately linearly with input volume across the 10, 20, 30, and 40-sample groups, even though all-vs-all pair counts grow quadratically
- JENSENSHANNON distance computation accounts for approximately 12.64% of total runtime on the 40-sample dataset
- Every dataset size is benchmarked under all three -s algorithms (COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON) with the default LOGARITHM weighting
- Both -m run modes (MAP default and REDUCE) are exercised and their elapsed times recorded separately
- Measurements are persisted in libra_runtime_benchmark.csv with one entry per dataset-size × algorithm × run-mode run

## Limitations

- Linear runtime scaling is demonstrated only for the synthetic 9.3 GB/sample datasets of 10–40 samples (93–372 GB); behavior at other scales is not established by this benchmark
- Scoring requires a Hadoop environment and a completed k-mer index; benchmark timing excludes preprocessing, so total end-to-end time will be larger
- Pair counts grow quadratically with sample count, so benchmark cost rises sharply even while per-volume runtime stays linear

## Evidence

- [readme] s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] m : run mode. MAP (default) | REDUCE: "m : run mode. MAP (default) | REDUCE"
- [abstract] Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset to enable global-scale analyses"
- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
