---
name: kbp-s-speed-metric-computation
description: Use when comparing processing speed (kilo-bases per second) of baseLess
  or competing MinION read-classification pipelines across low-power single-board
  hardware (Jetson Nano) and desktop configurations.
when_to_use_negative:
- The goal is classification accuracy (species identity calls), not inference throughput
  — use the accuracy benchmark skill instead
- Input is already basecalled FASTQ/aligned data rather than raw MinION fast5 electrical
  signal, since kbp/s here measures signal-space detection speed
- Evaluating model training/compilation time (compile_model, run_production_pipeline)
  rather than inference throughput
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: baseLess
  role: Tool being benchmarked; run_inference with --inference-mode once over 1000-read
    fast5 sets, timed for 10 repetitions on both hardware configs
  repo: https://github.com/cvdelannoy/baseLess
- name: docker
  role: Containerizes baseLess on the Jetson Nano (Dockerfile_jetson) with memory
    caps of 512MB on the 2GB model
- name: guppy
  role: 'Baseline competitor: guppy basecalling followed by minimap2 alignment, benchmarked
    at 2.0 kbp/s on the Nano'
- name: MinION
  role: Instrument producing the raw fast5 signal reads that all benchmarked pipelines
    process
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:02912926cd5895be93e28f02cbfd367629ec6c0874dd70243ef6e5a1cc9318bb
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
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# kbp-s-speed-metric-computation

## Summary

Computes per-tool inference throughput in kbp/s for baseLess and competing raw-signal MinION pipelines (Guppy+minimap2, SquiggleNet, DeepNano+minimap2, UNCALLED) benchmarked on a low-power Jetson Nano and a high-end desktop, the key deployment metric for a basecalling-free species detector.

## When to use

Use when comparing processing speed (kilo-bases per second) of baseLess or competing MinION read-classification pipelines across low-power single-board hardware (Jetson Nano) and desktop configurations.

## When NOT to use

- The goal is classification accuracy (species identity calls), not inference throughput — use the accuracy benchmark skill instead
- Input is already basecalled FASTQ/aligned data rather than raw MinION fast5 electrical signal, since kbp/s here measures signal-space detection speed
- Evaluating model training/compilation time (compile_model, run_production_pipeline) rather than inference throughput

## Inputs

- Directory of single-read fast5 files (1000-read sets, raw MinION signal)
- Compiled baseLess model (.h5, e.g. 16S read_detection or abundance model)
- Jetson Nano (2GB, docker+nvidia runtime) and desktop hardware configurations
- Competing pipeline outputs (guppy basecalls + minimap2, SquiggleNet, DeepNano+minimap2, UNCALLED)

## Outputs

- speed_benchmark_kbps.csv — one kbp/s entry per tool per hardware configuration, aggregated over 10 repetitions

## How to apply

For the Jetson Nano, build the baseLess container with `docker build -f Dockerfile_jetson -t baseless .` and run it with `--runtime nvidia`; on the 2GB Nano limit baseLess to 256MB (`--mem 256`) and the docker container to 512MB (`-m 512m`) to fit memory. Run `run_inference --fast5-in <dir> --model <model>.h5 --inference-mode once` on the same 1000-read fast5 sets for 10 timed repetitions, and repeat the protocol for Guppy+minimap2, SquiggleNet, DeepNano+minimap2, and UNCALLED; repeat all steps on the desktop configuration without docker memory caps. Convert read counts and total bases into kbp/s per tool per hardware configuration and aggregate across repetitions. Validate against the paper-reported speeds: baseLess at 5.0 kbp/s on the Nano (vs 2.0 for Guppy+minimap2) and 210 kbp/s on desktop (SquiggleNet 420, DeepNano+minimap2 120, UNCALLED 84).

## Related tools

- **baseLess** (Tool being benchmarked; run_inference with --inference-mode once over 1000-read fast5 sets, timed for 10 repetitions on both hardware configs) — https://github.com/cvdelannoy/baseLess
- **docker** (Containerizes baseLess on the Jetson Nano (Dockerfile_jetson) with memory caps of 512MB on the 2GB model)
- **guppy** (Baseline competitor: guppy basecalling followed by minimap2 alignment, benchmarked at 2.0 kbp/s on the Nano)
- **MinION** (Instrument producing the raw fast5 signal reads that all benchmarked pipelines process)

## Examples

```
baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5 --inference-mode once
```

## Evaluation signals

- speed_benchmark_kbps.csv contains positive kbp/s values for all 5 tools on both hardware configurations
- baseLess Nano value matches the paper-reported 5.0 kbp/s (more than twice Guppy+minimap2's 2.0 kbp/s)
- baseLess desktop value matches the paper-reported 210 kbp/s, below SquiggleNet (420) but above DeepNano+minimap2 (120) and UNCALLED (84)
- Each kbp/s figure derives from 10 timed repetitions over the same 1000-read fast5 sets, consistently aggregated
- Jetson Nano runs complete under the memory caps (baseLess --mem 256, docker -m 512m) without OOM termination

## Limitations

- The 2GB Jetson Nano requires separate memory limits on docker (-m 512m) and baseLess (--mem 256); without them the run can exhaust the 2GB of available memory
- Reported speeds are hardware-configuration-specific and should not be extrapolated to other single-board computers or GPUs
- Read-detection mode is attuned to 16S sequence, so throughput numbers assume 16S-targeted models and pre-generated k-mer networks (<1MB)

## Evidence

- [abstract] However, translating the electrical signal it outputs into a sequence of bases still requires mid-tier computer hardware: "However, translating the electrical signal it outputs into a sequence of bases still requires mid-tier computer hardware"
- [intro] it runs on inexpensive computational hardware like single-board computers: "it runs on inexpensive computational hardware like single-board computers"
- [methods] We found that the 2GB model still runs well with baseLess limited at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`).: "We found that the 2GB model still runs well with baseLess limited at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`)."
- [readme] The Nvidia Jetson Nano is a single-board computer with dedicated GPU. BaseLess is designed to be light enough to run on such low-powered hardware.: "The Nvidia Jetson Nano is a single-board computer with dedicated GPU. BaseLess is designed to be light enough to run on such low-powered hardware."
