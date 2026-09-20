---
name: single-board-hardware-performance-profiling
description: Use when measuring or comparing raw-signal target-detection throughput
  (kbp/s) of nanopore pipelines on memory-constrained single-board hardware such as
  the 2GB Nvidia Jetson Nano, or when validating that a tool is light enough for low-powered
  deployment.
when_to_use_negative:
- The analysis goal is classification accuracy (e.g. fish-species identity or 20-species
  mixture classification) rather than processing throughput on constrained hardware
- Input is already basecalled sequence (FASTQ) rather than raw fast5 signal data —
  baseLess operates directly on the electrical signal, so benchmarking it on basecalls
  is out of scope
- The tool under test explicitly requires mid-tier desktop hardware or cloud resources,
  making single-board profiling (256MB/512MB memory caps) inapplicable
tools:
- name: baseLess
  role: Tool under benchmark; runs k-mer neural-network target detection directly
    on raw MinION fast5 signals and is timed in kbp/s on Jetson Nano and desktop
  repo: https://github.com/cvdelannoy/baseLess
- name: docker
  role: Containerizes baseLess for the Jetson Nano via Dockerfile_jetson, with container
    memory capped at 512MB (`-m 512m`)
- name: guppy
  role: Oxford Nanopore basecaller used as a speed-comparison baseline (guppy basecalling
    followed by minimap2 alignment)
- name: MinION
  role: Portable nanopore sequencer whose raw fast5 output constitutes the benchmark
    input reads
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:95722f1dd3bc0d6e9493453f2ba58ec5d13f84455b61734ccf51a5ca8431e918
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-board-hardware-performance-profiling

## Summary

A benchmarking protocol for measuring inference processing speed (kbp/s) of baseLess and competing nanopore signal-classification pipelines on the memory-constrained Nvidia Jetson Nano (2GB) versus a high-end desktop, demonstrating that raw-signal target detection runs on single-board computers.

## When to use

Use when measuring or comparing raw-signal target-detection throughput (kbp/s) of nanopore pipelines on memory-constrained single-board hardware such as the 2GB Nvidia Jetson Nano, or when validating that a tool is light enough for low-powered deployment.

## When NOT to use

- The analysis goal is classification accuracy (e.g. fish-species identity or 20-species mixture classification) rather than processing throughput on constrained hardware
- Input is already basecalled sequence (FASTQ) rather than raw fast5 signal data — baseLess operates directly on the electrical signal, so benchmarking it on basecalls is out of scope
- The tool under test explicitly requires mid-tier desktop hardware or cloud resources, making single-board profiling (256MB/512MB memory caps) inapplicable

## Inputs

- Directories of single-read fast5 files (e.g. 1000-read fast5 sets) containing raw MinION electrical signals
- A compiled baseLess model file (.h5), e.g. an abundance- or read_detection-type model
- A Jetson Nano (2GB) with the Dockerfile_jetson docker image built, and a desktop reference machine
- Competing pipeline installations: guppy + minimap2, SquiggleNet, DeepNano + minimap2, UNCALLED

## Outputs

- speed_benchmark_kbps.csv: a table of processing speed (kbp/s) per tool per hardware configuration, aggregated over 10 repetitions
- Per-configuration timing measurements over 1000 reads per repetition

## How to apply

Build the Jetson Nano container image from the baseLess package with `docker build -f Dockerfile_jetson -t baseless .` and launch it interactively with `docker run -it --rm --runtime nvidia --network host baseless`. Because the 2GB Nano has little memory, cap baseLess at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`); optionally add `--oom-kill-disable` and stop the GUI (`sudo systemctl stop lxdm`) to free memory. Run baseLess inference with `run_inference --fast5-in <fast5 directory> --model <model>.h5 --inference-mode once`, timing 10 repetitions over 1000 reads, and run the comparison pipelines (guppy basecalling + minimap2, SquiggleNet, DeepNano+minimap2, UNCALLED) on the same 1000-read fast5 sets with the same protocol. Repeat the identical protocol on the desktop configuration without docker memory caps, convert read counts and total bases into processing speed (kbp/s) per tool per hardware configuration, aggregate across repetitions, and write the result to speed_benchmark_kbps.csv.

## Related tools

- **baseLess** (Tool under benchmark; runs k-mer neural-network target detection directly on raw MinION fast5 signals and is timed in kbp/s on Jetson Nano and desktop) — https://github.com/cvdelannoy/baseLess
- **docker** (Containerizes baseLess for the Jetson Nano via Dockerfile_jetson, with container memory capped at 512MB (`-m 512m`))
- **guppy** (Oxford Nanopore basecaller used as a speed-comparison baseline (guppy basecalling followed by minimap2 alignment))
- **MinION** (Portable nanopore sequencer whose raw fast5 output constitutes the benchmark input reads)

## Examples

```
docker run -it --rm --runtime nvidia --network host -m 512m baseless
```

## Evaluation signals

- speed_benchmark_kbps.csv contains an entry for each of the 5 tools (baseLess, Guppy+minimap2, SquiggleNet, DeepNano+minimap2, UNCALLED) on both hardware configurations with positive kbp/s values
- baseLess kbp/s values are consistent with paper-reported speeds: 5.0 kbp/s on the Jetson Nano and 210 kbp/s on desktop
- On the Jetson Nano, baseLess (5.0 kbps) is more than twice as fast as Guppy+minimap2 (2.0 kbps)
- Each reported speed is aggregated over 10 repetitions on the same 1000-read fast5 set per hardware configuration
- The Jetson Nano run completes without out-of-memory termination under the 256MB baseLess / 512MB docker memory caps

## Limitations

- The Jetson Nano has little memory to spare (2GB or 4GB depending on model), so docker and baseLess memory must be limited separately (baseLess --mem 256, container -m 512m), the docker OOM killer may need disabling, and the GUI should be switched off
- baseLess incurs docker-container overhead on the Nano; future versions plan to avoid the docker container entirely for less overhead
- Read-detection mode has been tested for and attuned to 16S sequence, so throughput results may not generalize to other target types
- Absolute speeds are hardware-dependent: baseLess reaches only 210 kbp/s on desktop versus 5.0 kbps on the Nano, while competitors such as SquiggleNet (420 kbps) are faster on desktop hardware

## Evidence

- [methods] We found that the 2GB model still runs well with baseLess limited at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`).: "We found that the 2GB model still runs well with baseLess limited at 256MB (`--mem 256`) and the docker container at 512MB (`-m 512m`)."
- [readme] BaseLess reduces your MinION sequencing device to a simple species detector. As a trade-off, it runs on inexpensive computational hardware like single-board computers.: "BaseLess reduces your MinION sequencing device to a simple species detector. As a trade-off, it runs on inexpensive computational hardware like single-board computers."
- [readme] The Nvidia Jetson Nano is a single-board computer with dedicated GPU. BaseLess is designed to be light enough to run on such low-powered hardware.: "The Nvidia Jetson Nano is a single-board computer with dedicated GPU. BaseLess is designed to be light enough to run on such low-powered hardware."
- [abstract] We show that baseLess can accurately determine the identity of reads between three closely related fish species and can classify sequences in mixtures of 20 bacterial species: "We show that baseLess can accurately determine the identity of reads between three closely related fish species and can classify sequences in mixtures of 20 bacterial species"
- [readme] Before first usage open a terminal in the baseless directory and build the image: docker build -f Dockerfile_jetson -t baseless .: [evidence span withheld — anchor status: not_found]
