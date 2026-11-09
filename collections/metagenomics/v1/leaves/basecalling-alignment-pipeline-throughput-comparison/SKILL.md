---
name: basecalling-alignment-pipeline-throughput-comparison
description: Use when comparing inference speed of nanopore raw-signal classifiers
  or basecalling pipelines across low-powered single-board computers and desktop hardware,
  and kbp/s throughput per tool is the target metric.
when_to_use_negative:
- The goal is classification accuracy or species-detection performance rather than
  processing-speed throughput comparison.
- The hardware under test is not a nanopore-capable compute target (no GPU or no docker/nvidia
  runtime available), since baseLess and the competing pipelines require it.
- Input is already basecalled sequence data (FASTQ/BAM) rather than raw fast5 signal
  files, which all benchmarked pipelines consume.
edam_topics:
- http://edamontology.org/topic_0623
tools:
- name: baseLess
  role: Tool under benchmark; runs raw-signal k-mer detection inference on fast5 reads
    on both hardware configurations
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Baseline basecaller; combined with minimap2 alignment to form the basecalling
    pipeline compared against baseLess
- name: docker
  role: Containerizes baseLess for reproducible execution on the Jetson Nano with
    memory limits
- name: MinION
  role: Nanopore sequencer producing the fast5 raw-signal reads used in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:3a1f51080a48be055ed3b2e0484cd3d9ea68e64e454897875928e9168aac086f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - MinION
  - baseLess
  - docker
  - guppy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# basecalling-alignment-pipeline-throughput-comparison

## Summary

Benchmarks the raw-signal processing throughput (kbp/s) of baseLess against basecalling-and-alignment pipelines (Guppy+minimap2, SquiggleNet, DeepNano+minimap2, UNCALLED) on edge hardware (Jetson Nano) and desktop GPUs, to quantify how much speed the no-basecalling approach buys.

## When to use

Use when comparing inference speed of nanopore raw-signal classifiers or basecalling pipelines across low-powered single-board computers and desktop hardware, and kbp/s throughput per tool is the target metric.

## When NOT to use

- The goal is classification accuracy or species-detection performance rather than processing-speed throughput comparison.
- The hardware under test is not a nanopore-capable compute target (no GPU or no docker/nvidia runtime available), since baseLess and the competing pipelines require it.
- Input is already basecalled sequence data (FASTQ/BAM) rather than raw fast5 signal files, which all benchmarked pipelines consume.

## Inputs

- Directory of 1000 fast5 reads (single-read fast5 files)
- Trained baseLess model (.h5)
- Jetson Nano (2GB) with docker and nvidia runtime
- Desktop hardware with GPU
- Comparison pipelines: guppy, minimap2, SquiggleNet, DeepNano, UNCALLED

## Outputs

- speed_benchmark_kbps.csv with one kbp/s entry per tool per hardware configuration (10 repetitions aggregated)

## How to apply

Run each pipeline on the same set of 1000 fast5 reads, timing 10 repetitions per tool per hardware configuration. On the Jetson Nano (2GB), build the baseLess image with `docker build -f Dockerfile_jetson -t baseless .`, launch with `--runtime nvidia`, and cap baseLess at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`); run `run_inference --inference-mode once` over the fast5 directory. Run the comparison pipelines on identical inputs: guppy basecalling followed by minimap2 alignment, SquiggleNet, DeepNano+minimap2, and UNCALLED. Repeat on the desktop configuration without docker memory caps. Convert read counts and total bases into processing speed (kbp/s) per tool per configuration and aggregate across repetitions; reference speeds from the paper are baseLess 5.0 kbp/s on Nano vs Guppy+minimap2 2.0 kbp/s, and on desktop SquiggleNet 420 kbp/s, baseLess 210 kbp/s, DeepNano+minimap2 120 kbp/s, UNCALLED 84 kbp/s.

## Related tools

- **baseLess** (Tool under benchmark; runs raw-signal k-mer detection inference on fast5 reads on both hardware configurations) — https://github.com/cvdelannoy/baseLess
- **guppy** (Baseline basecaller; combined with minimap2 alignment to form the basecalling pipeline compared against baseLess)
- **docker** (Containerizes baseLess for reproducible execution on the Jetson Nano with memory limits)
- **MinION** (Nanopore sequencer producing the fast5 raw-signal reads used in the benchmark)

## Examples

```
run_inference --fast5-in <fast5 directory> --model <model>.h5 --inference-mode once
```

## Evaluation signals

- speed_benchmark_kbps.csv contains an entry for each of the 5 tools on both hardware configurations with positive kbp/s values.
- baseLess Jetson Nano throughput is ~5.0 kbp/s and desktop throughput is ~210 kbp/s, consistent with paper-reported speeds.
- Guppy+minimap2 on Jetson Nano yields ~2.0 kbp/s, i.e. less than half of baseLess, confirming the reported >2x speedup.
- All 10 repetitions completed per tool per configuration; read counts convert to total bases without loss.
- On the Jetson Nano run, baseLess ran within the 256MB limit and the docker container within 512MB without OOM termination.

## Limitations

- The 2GB Jetson Nano requires careful memory management (baseLess --mem 256, docker -m 512m) and GUI disabling to avoid out-of-memory failures.
- baseLess runs indefinitely unless --inference-mode once is used, so benchmark timing must ensure the run terminates after the fixed read set.
- baseLess read_detection has been attuned to 16S sequence; throughput numbers may not generalize to other targets.

## Evidence

- [results] baseLess processed 5.0 kb per second (kbps), more than twice as fast as Guppy+minimap2 (2.0 kbps): [evidence span withheld — anchor status: not_found]
- [results] desktop pipeline speeds: [evidence span withheld — anchor status: not_found]
- [readme] Jetson Nano docker memory limits: "We found that the 2GB model still runs well with baseLess limited at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`)."
- [readme] Jetson Nano docker build/run: [evidence span withheld — anchor status: not_found]
- [readme] inference mode for bounded runs: "To run on a directory of reads and stop when all reads are analysed, provide `--inference-mode once`."
