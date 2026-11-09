---
name: runtime-memory-benchmark-profiling
description: Use when benchmarking taxonomic binning or profiling tools on the same
  read samples and databases and you need comparable per-tool runtime and peak-memory
  figures alongside F1-scores.
when_to_use_negative:
- You only need classification accuracy (F1/L1-norm) and are not comparing resource
  consumption across tools
- The classification runs are not wrapped with a timing harness or executed via the
  Snakemake pipeline, so .time/.stats outputs cannot be produced consistently
- Samples are real (non-simulated) reads without the ground-truth files needed for
  the paired AMBER evaluation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ganon2
  role: Classification tool whose runtime and peak memory are benchmarked against
    competitors on the 16 CAMI samples
  repo: https://github.com/pirovc/ganon
- name: Snakemake
  role: Workflow engine running the build/classify steps with --use-conda; its benchmark
    records raw run metrics (*.bench.tsv)
  repo: https://github.com/pirovc/ganon_benchmark
- name: AMBER
  role: Produces the *.rank.tsv evaluation tables whose species-level f1s are joined
    with the timing/memory statistics
  repo: https://github.com/CAMI-challenge/AMBER
- name: MetaBench
  role: Alternative pipeline that continuously benchmarks build, binning and profiling,
    emitting *.build.bench./tsv metrics with optional repeats
  repo: https://github.com/pirovc/metabench
- name: kraken2+bracken
  role: Competing profiling tool benchmarked for runtime/memory (RefSeq RG runtime
    ~59 min for all 16 samples)
- name: kmcp
  role: Competing classifier benchmarked for runtime/memory (RefSeq RG runtime ~4
    h 47 min for all 16 samples)
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:04eff2a63395904cfa39fa0296fe295f736a24c27a2dbaf278567807267030f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pirovc/ganon
  tools:
  - AMBER
  - MetaBench
  - Snakemake
  - conda
  - ganon2
  - kmcp
  - kraken2+bracken
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# runtime-memory-benchmark-profiling

## Summary

Reproduce the runtime and peak-memory benchmarking of metagenomic classifiers (ganon2, kraken2+bracken, kmcp) by capturing '/usr/bin/time -v' output for every tool-sample-database classification run and aggregating wall-clock time and maximum resident set size across the 16 simulated CAMI samples and RefSeq configurations.

## When to use

Use when benchmarking taxonomic binning or profiling tools on the same read samples and databases and you need comparable per-tool runtime and peak-memory figures alongside F1-scores.

## When NOT to use

- You only need classification accuracy (F1/L1-norm) and are not comparing resource consumption across tools
- The classification runs are not wrapped with a timing harness or executed via the Snakemake pipeline, so .time/.stats outputs cannot be produced consistently
- Samples are real (non-simulated) reads without the ground-truth files needed for the paired AMBER evaluation

## Inputs

- 16 simulated short-read .fq.gz samples from CAMI 1+2 challenge (files/reads/ or data.cami-challenge.org)
- Pre-built tool indices for each RefSeq configuration (ganon2, kraken2+bracken, kmcp)
- Snakemake classify pipeline (classify/Snakefile) with files/config/classify_toy_taxid.yaml
- AMBER evaluation *.rank.tsv files for joining timing with F1-scores

## Outputs

- .time files with '/usr/bin/time -v' wall-clock output per tool-sample-database run
- .stats files with peak memory (Maximum resident set size) per run
- Aggregated results table (e.g. profiling_benchmark_results.tsv) with per-tool runtime, peak memory, and species-level F1 medians

## How to apply

Wrap each classification invocation (e.g. ganon classify, kraken2, kraken2+bracken in default profiling mode with the minimum-abundance threshold) with '/usr/bin/time -v' so elapsed wall time is written to a .time file and peak memory ('Maximum resident set size') to a .stats file, for each sample-database combination (RefSeqOLD, RefSeqCG, RefSeqCG-top-3, RefSeqALL, RefSeqALL-top-3, full RefSeq). Drive the runs with the Snakemake classify pipeline (classify/Snakefile, --use-conda --cores 48 -k) using files/config/classify_toy_taxid.yaml, so timing is captured consistently for all 16 simulated short-read samples from CAMI 1+2. Optionally run with repeat > 1 in a MetaBench-style config and select the fastest run to reduce noise. After AMBER evaluation produces the *.rank.tsv files, join the per-tool runtime/memory values with the species-level f1s values and aggregate into a results table such as profiling_benchmark_results.tsv.

## Related tools

- **ganon2** (Classification tool whose runtime and peak memory are benchmarked against competitors on the 16 CAMI samples) — https://github.com/pirovc/ganon
- **Snakemake** (Workflow engine running the build/classify steps with --use-conda; its benchmark records raw run metrics (*.bench.tsv)) — https://github.com/pirovc/ganon_benchmark
- **AMBER** (Produces the *.rank.tsv evaluation tables whose species-level f1s are joined with the timing/memory statistics) — https://github.com/CAMI-challenge/AMBER
- **MetaBench** (Alternative pipeline that continuously benchmarks build, binning and profiling, emitting *.build.bench./tsv metrics with optional repeats) — https://github.com/pirovc/metabench
- **kraken2+bracken** (Competing profiling tool benchmarked for runtime/memory (RefSeq RG runtime ~59 min for all 16 samples))
- **kmcp** (Competing classifier benchmarked for runtime/memory (RefSeq RG runtime ~4 h 47 min for all 16 samples))

## Examples

```
snakemake -s classify/Snakefile --configfile files/config/classify_toy_taxid.yaml --cores 48 --use-conda -k  # classify runs wrapped with /usr/bin/time -v producing .time/.stats
```

## Evaluation signals

- RefSeq RG configuration runtimes for all 16 samples should match reported values: ~43 min (ganon2), ~59 min (kraken2+bracken), ~4 h 47 min (kmcp)
- Peak-memory values extracted from .stats files should be consistent with Table 4 of the manuscript
- Every tool-sample-database run must have both a .time and a .stats file; missing files indicate the '/usr/bin/time -v' wrapper was bypassed
- Per-tool median species-level F1-scores joined with timings should match validation targets (binning: ganon2 ~0.77, kraken2 ~0.61, Metacache ~0.45 on full RefSeq)
- If repeat > 1 is used, only the fastest run's benchmark metrics should be retained in the standardized JSON output

## Limitations

- Timing is wall-clock on a shared 48-core machine; concurrent load can skew runtime comparisons between tools
- kmcp was only run 'where feasible' in binning mode, so runtime/memory coverage may be incomplete for some database configurations
- Timing figures are specific to the simulated CAMI 1+2 samples and RefSeq configurations and may not transfer to real reads or other databases

## Evidence

- [readme] runtime and peak-memory from the .stats files: [evidence span withheld — anchor status: not_found]
- [other] RefSeq RG runtimes targets: [evidence span withheld — anchor status: not_found]
- [other] recording time and stats: [evidence span withheld — anchor status: not_found]
- [abstract] ganon2 fastest: "ganon2 is one of the fastest tools evaluated and enables the use of larger, more diverse, and up-to-date reference sets"
- [readme] Snakemake benchmark metrics: [evidence span withheld — anchor status: not_found]
