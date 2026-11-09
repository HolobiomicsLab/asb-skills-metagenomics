---
name: wall-clock-runtime-benchmarking
description: Use when reproducing or benchmarking reported execution times of a Snakemake/SBATCH
  pipeline per dataset, e.g. to validate Mapler's 2h4m/3h46m/5h3m figures under a
  48-thread, 200G allocation.
when_to_use_negative:
- The goal is assembly or binning quality (completeness/contamination, mapped read
  ratios), not runtime measurement — use Mapler's evaluation outputs instead.
- Only a quick installation check is needed — the README's test dataset with 16 CPUs
  / 15G RAM (~15 minutes) is the appropriate lightweight path, not the full three-dataset
  benchmark.
- Runs use different thread/memory allocations than the reported 48 CPU / 200G, making
  wall-clock times non-comparable to the reference durations.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Mapler
  role: The Snakemake assembly-and-evaluation pipeline whose per-dataset wall-clock
    runtime is benchmarked
  repo: https://github.com/Nimauric/Mapler
- name: Snakemake
  role: Workflow engine (>=8.28) whose logs and rule outputs provide the timing measurements
  repo: https://github.com/Nimauric/Mapler
- name: snakemake-executor-plugin-slurm
  role: SLURM executor plugin enabling 'sbatch pipeline.sh <configfile>' cluster launches
    with fixed resources
  repo: https://github.com/Nimauric/Mapler
- name: metaMDBG
  role: One of the three assemblers run per dataset whose per-run duration contributes
    to the measured wall-clock time
  repo: https://github.com/GaetanBenoitDev/metaMDBG
- name: metaflye
  role: Assembler run per dataset in the timed configuration
  repo: https://github.com/mikolmogorov/Flye
- name: hifiasm-meta
  role: Assembler run per dataset in the timed configuration
  repo: https://github.com/xfengnefx/hifiasm-meta
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:55aae48cd81ee8fd81c1ba1d5aa4dfc6d0983fb3aa517e8c0337bca310f42452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nimauric/Mapler
  tools:
  - Conda
  - GTDB-Tk
  - Mapler
  - MetaQUAST
  - Snakemake
  - hifiasm-meta
  - metaMDBG
  - metaflye
  - minimap2
  - snakemake-executor-plugin-slurm
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# wall-clock-runtime-benchmarking

## Summary

Benchmark the wall-clock runtime of the Mapler Snakemake pipeline (assembly with metaMDBG, metaflye, hifiasm-meta plus evaluation) on the Mock community, Gut microbiome, and Deadwood HiFi datasets under a fixed 48 CPU / 200G SLURM allocation, and compare measured durations against reference values (2 h 4 min, 3 h 46 min, 5 h 3 min).

## When to use

Use when reproducing or benchmarking reported execution times of a Snakemake/SBATCH pipeline per dataset, e.g. to validate Mapler's 2h4m/3h46m/5h3m figures under a 48-thread, 200G allocation.

## When NOT to use

- The goal is assembly or binning quality (completeness/contamination, mapped read ratios), not runtime measurement — use Mapler's evaluation outputs instead.
- Only a quick installation check is needed — the README's test dataset with 16 CPUs / 15G RAM (~15 minutes) is the appropriate lightweight path, not the full three-dataset benchmark.
- Runs use different thread/memory allocations than the reported 48 CPU / 200G, making wall-clock times non-comparable to the reference durations.

## Inputs

- HiFi long-read FASTQ samples: Mock community, Gut microbiome, Deadwood (doi:10.57745/2SA8AB)
- config/config_template.yaml used to create one per-dataset Snakemake configfile
- SLURM cluster with sbatch and the snakemake-executor-plugin-slurm

## Outputs

- outputs/<sample_name>/<assembler>/assembly.fasta for metaMDBG, metaflye, hifiasm-meta
- outputs/<sample_name>/<assembler>/reads_on_contigs_mapping_evaluation/report.txt
- Per-sample and per-assembler wall-clock durations extracted from logs/<analysis_name>/<date_hour> and artifact timestamps
- execution_time_comparison.csv tabulating measured vs reference durations (2 h 4 min, 3 h 46 min, 5 h 3 min)

## How to apply

Clone the Mapler repository and build the 'mapler' conda environment with bioconda::snakemake>=8.28, conda-forge::conda>=24.1.2, and bioconda::snakemake-executor-plugin-slurm. Copy config/config_template.yaml into one configfile per dataset, listing the sample reads (Mock community, Gut microbiome, Deadwood from doi:10.57745/2SA8AB), enabling metaMDBG, metaflye, and hifiasm-meta, and turning on checkm and read_mapping_evaluation. Set resource parameters to 48 threads and 200G memory per rule so the runtime is directly comparable to the reference, optionally pre-build conda environments with 'snakemake --use-conda --conda-create-envs-only -c1', then launch each run with 'sbatch pipeline.sh <configfile>'. Extract per-sample and per-assembler wall-clock durations from logs/<analysis_name>/<date_hour> and the timestamps of outputs/<sample_name>/<assembler>/ artifacts, tabulate them against 2 h 4 min, 3 h 46 min, and 5 h 3 min, and write execution_time_comparison.csv.

## Related tools

- **Mapler** (The Snakemake assembly-and-evaluation pipeline whose per-dataset wall-clock runtime is benchmarked) — https://github.com/Nimauric/Mapler
- **Snakemake** (Workflow engine (>=8.28) whose logs and rule outputs provide the timing measurements) — https://github.com/Nimauric/Mapler
- **snakemake-executor-plugin-slurm** (SLURM executor plugin enabling 'sbatch pipeline.sh <configfile>' cluster launches with fixed resources) — https://github.com/Nimauric/Mapler
- **metaMDBG** (One of the three assemblers run per dataset whose per-run duration contributes to the measured wall-clock time) — https://github.com/GaetanBenoitDev/metaMDBG
- **metaflye** (Assembler run per dataset in the timed configuration) — https://github.com/mikolmogorov/Flye
- **hifiasm-meta** (Assembler run per dataset in the timed configuration) — https://github.com/xfengnefx/hifiasm-meta

## Examples

```
snakemake --use-conda --conda-create-envs-only -c1 --configfile config/config_mock.yaml; sbatch pipeline.sh config/config_mock.yaml
```

## Evaluation signals

- Each run completed: outputs/<sample_name>/<assembler>/assembly.fasta and reads_on_contigs_mapping_evaluation/report.txt both exist for all three assemblers and samples.
- Measured wall-clock durations are consistent with the reference values of 2 h 4 min (Mock community), 3 h 46 min (Gut microbiome), and 5 h 3 min (Deadwood) under the 48 CPU / 200G allocation.
- Resource parameters in each configfile match the reference setting of 48 threads and 200G memory per rule.
- execution_time_comparison.csv contains one row per dataset with both measured and reference durations.
- Duration ordering is preserved across datasets (Deadwood slowest, Mock community fastest), reflecting dataset size differences.

## Limitations

- Wall-clock times depend on cluster load and the 48 CPU / 200G allocation; runs under different resource settings are not comparable to the reference durations.
- Dependency installation time is excluded from the reference timings; pre-building conda environments with --conda-create-envs-only is recommended to avoid skewing the first run.
- Only part of the analysis runs on the built-in test dataset, so test runs cannot be used to reproduce the full per-dataset runtimes.

## Evidence

- [methods] Clone the Mapler repository and create the 'mapler' conda environment with bioconda::snakemake>=8.28: [evidence span withheld — anchor status: not_found]
- [results] Mapler's evaluation of three assemblers took 2 h 4 min on the Mock community: [evidence span withheld — anchor status: not_found]
- [readme] SLURM executor plugin needed for cluster execution: "If running on a SLURM cluster, the Snakemake SLURM executor plugin is also needed."
- [readme] Resource parameters configure memory, threads, and execution time: "resource parameters allow users to configure memory, threads, and execution time"
- [methods] Set resource parameters to 48 threads and 200G memory per rule: [evidence span withheld — anchor status: not_found]
