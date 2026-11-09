---
name: synthetic-metagenome-dataset-generation
description: Use when you need to benchmark Libra's runtime scaling on synthetic metagenome
  inputs of known size, or to tabulate per-metric (Cosine Similarity, Bray-Curtis,
  Jensen-Shannon) runtime shares under MAP and REDUCE Hadoop modes.
when_to_use_negative:
- Input consists of real biological datasets (e.g. 48 HMP 16S/WGS samples, CAMI toy
  PacBio data, or 43 Tara Ocean Viromes) rather than size-controlled synthetic samples
  for scaling tests
- The goal is accuracy or clustering-quality validation of the distance metric rather
  than runtime/scalability measurement
- Datasets are small enough that Hadoop MapReduce startup overhead dominates runtime,
  making linear-scaling conclusions meaningless
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Libra
  role: Hadoop MapReduce tool whose `preprocess` and `core` commands index the synthetic
    samples and run all-vs-all scoring for the timing benchmark
  repo: https://github.com/iychoi/libra
- name: Hadoop
  role: Execution framework providing the MAP and REDUCE run modes (-m option) under
    which elapsed times are recorded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:25571dcf00f79620d06f2dd2044d7b20cdfc0e9ebee9c840551b04a6c516730e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Hadoop
  - Libra
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# synthetic-metagenome-dataset-generation

## Summary

Generates synthetic benchmark metagenome datasets of controlled size (9.3 GB per sample, grouped at 10/20/30/40 samples for 93–372 GB totals) to measure how Libra's Hadoop-based all-vs-all k-mer comparison runtime scales with input volume.

## When to use

Use when you need to benchmark Libra's runtime scaling on synthetic metagenome inputs of known size, or to tabulate per-metric (Cosine Similarity, Bray-Curtis, Jensen-Shannon) runtime shares under MAP and REDUCE Hadoop modes.

## When NOT to use

- Input consists of real biological datasets (e.g. 48 HMP 16S/WGS samples, CAMI toy PacBio data, or 43 Tara Ocean Viromes) rather than size-controlled synthetic samples for scaling tests
- The goal is accuracy or clustering-quality validation of the distance metric rather than runtime/scalability measurement
- Datasets are small enough that Hadoop MapReduce startup overhead dominates runtime, making linear-scaling conclusions meaningless

## Inputs

- Synthetic FASTA/FASTQ metagenome sample files sized at 9.3 GB per sample
- Four sample groups of 10, 20, 30, and 40 samples (93–372 GB total)
- Libra k-mer index directories produced by `preprocess` from the synthetic groups

## Outputs

- Per-run score directories produced via the -o option for each dataset size, scoring algorithm, and run mode
- Runtime table grouped by dataset size and scoring algorithm (COSINESIMILARITY, BRAYCURTIS, JENSENSHANNON; MAP and REDUCE modes)
- libra_runtime_benchmark.csv containing elapsed times and the Jensen-Shannon runtime share (~12.64%) for the 40-sample dataset

## How to apply

Generate synthetic metagenome samples at 9.3 GB each and organize them into four groups containing 10, 20, 30, and 40 samples (93–372 GB total). Preprocess each group to build a k-mer index (e.g. hadoop jar libra-all.jar preprocess with -k 20), then run Libra scoring via `core` with the -s option across all three algorithms (COSINESIMILARITY default, BRAYCURTIS, JENSENSHANNON), keeping the default LOGARITHM weighting (-w) and directing output to a score directory with -o. Exercise both Hadoop run modes via -m (MAP default and REDUCE) and record elapsed time per run, noting that the number of all-vs-all sample pairs grows quadratically with sample count. Tabulate runtimes by dataset size and scoring algorithm, compute the Jensen-Shannon share of runtime on the 40-sample dataset (expected ~12.64%), and save measurements to libra_runtime_benchmark.csv. Accept the benchmark only if runtime scales approximately linearly with input volume across the 10–40 sample groups.

## Related tools

- **Libra** (Hadoop MapReduce tool whose `preprocess` and `core` commands index the synthetic samples and run all-vs-all scoring for the timing benchmark) — https://github.com/iychoi/libra
- **Hadoop** (Execution framework providing the MAP and REDUCE run modes (-m option) under which elapsed times are recorded)

## Examples

```
hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir
```

## Evaluation signals

- Runtime increases approximately linearly with input volume across the 10, 20, 30, and 40-sample groups even though all-vs-all sample pairs grow quadratically
- Jensen-Shannon distance computation accounts for approximately 12.64% of total runtime on the 40-sample dataset
- Runs complete for all three scoring algorithms (-s COSINESIMILARITY | BRAYCURTIS | JENSENSHANNON) under both -m MAP and -m REDUCE modes
- libra_runtime_benchmark.csv contains one row per dataset-size × scoring-algorithm × run-mode combination with recorded elapsed times

## Limitations

- The number of all-vs-all sample pairs grows quadratically with sample count, so the linear runtime scaling claim depends on Hadoop distributing pair computations
- Benchmark validity is tied to the paper's sizing (9.3 GB per sample, 93–372 GB totals); substantially different group sizes or k-mer settings (e.g. -k 20, NOTUNIQUE kmer filter default) may change runtime behavior
- Requires a Hadoop-enabled environment; Libra is executed as a Java jar (`libra-all.jar`), so results depend on cluster configuration and reducer/task counts

## Evidence

- [methods] Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON: "Scoring Options - s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
- [readme] hadoop jar libra-all.jar core -w LOGARITHM -o /score_dir /index_dir: [evidence span withheld — anchor status: external_doc]
- [abstract] Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset: "Libra uses all of the reads to calculate k-mer abundance in a Hadoop architecture that can scale to any size dataset"
- [intro] the simulated Critical Assessment of Metagenome Interpretation (CAMI) “toy” Pacific Biosciences (PacBio) dataset: "the simulated Critical Assessment of Metagenome Interpretation (CAMI) “toy” Pacific Biosciences (PacBio) dataset"
