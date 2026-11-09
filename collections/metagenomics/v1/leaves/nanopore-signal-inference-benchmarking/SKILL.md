---
name: nanopore-signal-inference-benchmarking
description: Use when you need to measure and compare inference/processing throughput
  (kbp/s) of raw MinION fast5 signal tools or basecalling pipelines across hardware
  tiers such as a 2GB Jetson Nano versus a high-end desktop.
when_to_use_negative:
- The goal is accuracy of species detection or classification (use the baseLess validation/accuracy
  evaluation instead of speed benchmarking)
- Input data are already basecalled FASTQ/FASTA rather than raw fast5 signal files,
  since baseLess and UNCALLED/SquiggleNet comparisons operate on raw electrical signal
- Hardware differs from the benchmarked configurations (2GB Jetson Nano or the paper's
  desktop) without re-running the protocol, since kbp/s values are hardware-specific
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: baseLess
  role: Target tool whose raw-signal inference speed (kbp/s) is measured on Jetson
    Nano and desktop, via run_inference in once-mode
  repo: https://github.com/cvdelannoy/baseLess
- name: docker
  role: Containerizes baseLess on the Jetson Nano (Dockerfile_jetson) with a 512MB
    memory cap (`-m 512m`) for reproducible low-power benchmarking
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls the same 1000-read fast5 sets as the first stage of the Guppy+minimap2
    comparison pipeline
- name: MinION
  role: Portable nanopore sequencer whose fast5 output (electrical signal reads) is
    the benchmark input data
- name: SquiggleNet
  role: Comparison pipeline benchmarked on identical 1000-read fast5 sets; fastest
    on desktop at 420 kbps
- name: UNCALLED
  role: Comparison pipeline benchmarked on identical read sets; 84 kbps on desktop
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:3d69075eb4dcfc2ee7860419efc678551e4f364fc1f676b643e5205cdc082290
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - MinION
  - SquiggleNet
  - UNCALLED
  - baseLess
  - docker
  - guppy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nanopore-signal-inference-benchmarking

## Summary

Benchmarks the wall-clock inference speed (kb processed per second) of raw-signal nanopore tools such as baseLess against basecalling+alignment pipelines (Guppy+minimap2, SquiggleNet, DeepNano+minimap2, UNCALLED) on low-power (Jetson Nano) and desktop hardware, following the baseLess evaluation protocol.

## When to use

Use when you need to measure and compare inference/processing throughput (kbp/s) of raw MinION fast5 signal tools or basecalling pipelines across hardware tiers such as a 2GB Jetson Nano versus a high-end desktop.

## When NOT to use

- The goal is accuracy of species detection or classification (use the baseLess validation/accuracy evaluation instead of speed benchmarking)
- Input data are already basecalled FASTQ/FASTA rather than raw fast5 signal files, since baseLess and UNCALLED/SquiggleNet comparisons operate on raw electrical signal
- Hardware differs from the benchmarked configurations (2GB Jetson Nano or the paper's desktop) without re-running the protocol, since kbp/s values are hardware-specific

## Inputs

- Directory of single-read MinION fast5 files (fixed set of 1000 reads shared across all tools)
- Trained baseLess .h5 model (e.g. abundance or read_detection model)
- Dockerfile_jetson from the baseLess repository for the Jetson Nano configuration
- Guppy basecalls plus a reference for minimap2 alignment (baseline pipeline)

## Outputs

- speed_benchmark_kbps.csv: one kbp/s entry per tool per hardware configuration, aggregated over 10 repetitions
- Per-tool timing measurements over 1000 reads on Jetson Nano and desktop configurations

## How to apply

Build the Jetson Nano container image (`docker build -f Dockerfile_jetson -t baseless .`) and run it with `--runtime nvidia`, limiting baseLess to 256MB (`--mem 256`) and the container to 512MB (`-m 512m`) as validated for the 2GB model. Run baseLess with `run_inference --inference-mode once` over the same fixed fast5 read sets (1000 reads) used for every competitor, timing 10 repetitions per tool. Benchmark comparison pipelines (guppy basecalling + minimap2 alignment, SquiggleNet, DeepNano+minimap2, UNCALLED) on identical 1000-read fast5 inputs with the same repetition protocol, then repeat everything on the desktop configuration without docker memory caps. Convert read counts and total bases into kbp/s per tool per hardware configuration and aggregate across repetitions. Rationale: identical inputs, repetition counts, and memory caps are what make the 5.0 kbp/s (Nano) and 210 kbp/s (desktop) baseLess figures comparable against competing tools.

## Related tools

- **baseLess** (Target tool whose raw-signal inference speed (kbp/s) is measured on Jetson Nano and desktop, via run_inference in once-mode) — https://github.com/cvdelannoy/baseLess
- **docker** (Containerizes baseLess on the Jetson Nano (Dockerfile_jetson) with a 512MB memory cap (`-m 512m`) for reproducible low-power benchmarking) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls the same 1000-read fast5 sets as the first stage of the Guppy+minimap2 comparison pipeline)
- **MinION** (Portable nanopore sequencer whose fast5 output (electrical signal reads) is the benchmark input data)
- **SquiggleNet** (Comparison pipeline benchmarked on identical 1000-read fast5 sets; fastest on desktop at 420 kbps)
- **UNCALLED** (Comparison pipeline benchmarked on identical read sets; 84 kbps on desktop)

## Examples

```
docker build -f Dockerfile_jetson -t baseless . ; baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5 --inference-mode once
```

## Evaluation signals

- speed_benchmark_kbps.csv contains an entry for each of the 5 tools on both hardware configurations with positive kbp/s values
- baseLess results are consistent with paper-reported speeds: 5.0 kbp/s on Jetson Nano and 210 kbp/s on desktop
- Relative ranking is reproduced: on Nano baseLess (5.0 kbps) more than doubles Guppy+minimap2 (2.0 kbps); on desktop SquiggleNet (420) > baseLess (210) > DeepNano+minimap2 (120) > UNCALLED (84)
- All tools are timed over the same 1000-read fast5 sets with 10 repetitions per tool per configuration
- Jetson Nano runs respect the validated memory caps (baseLess --mem 256, docker -m 512m) without out-of-memory failures

## Limitations

- kbp/s values are hardware-specific; figures from the Jetson Nano and the paper's desktop are not transferable to other machines without re-benchmarking
- On desktop hardware baseLess (210 kbps) is slower than SquiggleNet (420 kbps); its advantage is specifically on inexpensive single-board computers
- The docker container adds overhead on the Jetson Nano; future versions of baseLess are planned to avoid docker entirely
- The 2GB Jetson Nano requires explicit memory limiting (baseLess --mem 256, docker -m 512m) to run reliably

## Evidence

- [methods] Jetson memory limits: "We found that the 2GB model still runs well with baseLess limited at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`)."
- [results] Reported benchmark speeds: [evidence span withheld — anchor status: not_found]
- [readme] Jetson Nano hardware rationale: "The Nvidia Jetson Nano is a single-board computer with dedicated GPU. BaseLess is designed to be light enough to run on such low-powered hardware."
- [readme] Batch inference mode: "To run on a directory of reads and stop when all reads are analysed, provide `--inference-mode once`."
