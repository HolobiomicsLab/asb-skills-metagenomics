---
name: pipeline-resource-configuration-tuning
description: Use when deploying the Mapler pipeline on HiFi metagenomic samples (e.g.
  Mock community, Gut microbiome, Deadwood) and you must set per-rule thread/memory
  allocations or compare measured wall-clock durations against reported reference
  times.
when_to_use_negative:
- The goal is interpreting assembly quality metrics (completeness/contamination, aligned
  read/base ratios) rather than configuring or timing the pipeline run itself
- You need to evaluate a pre-existing assembly without running the full pipeline —
  use config_test_evaluation_only.yaml-style evaluation-only mode instead of full
  resource tuning
- The environment is not SLURM-based; use ./local_pipeline.sh with local resources
  rather than sbatch-based SLURM configuration
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow engine whose configfile resource parameters set memory, threads,
    and execution time per rule
  repo: https://github.com/Nimauric/Mapler
- name: Conda
  role: Manages the mapler environment and pipeline dependencies, which can be pre-created
    with --conda-create-envs-only
  repo: https://github.com/Nimauric/Mapler
- name: snakemake-executor-plugin-slurm
  role: SLURM executor plugin enabling cluster submission via sbatch pipeline.sh <configfile>
  repo: https://github.com/Nimauric/Mapler
- name: Mapler
  role: Pipeline whose YAML config exposes functional and resource parameters that
    are tuned per dataset
  repo: https://github.com/Nimauric/Mapler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:3a3f674d199f8d235cbf56d0ed669d77b6bd38bfe3d2523e7d4eb633d36086b5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# pipeline-resource-configuration-tuning

## Summary

Configure and tune the Mapler Snakemake pipeline's computational resources (threads, memory, execution time) per rule via its YAML configfile, so assembly and evaluation of HiFi metagenome samples completes reliably on a SLURM cluster with predictable wall-clock times.

## When to use

Use when deploying the Mapler pipeline on HiFi metagenomic samples (e.g. Mock community, Gut microbiome, Deadwood) and you must set per-rule thread/memory allocations or compare measured wall-clock durations against reported reference times.

## When NOT to use

- The goal is interpreting assembly quality metrics (completeness/contamination, aligned read/base ratios) rather than configuring or timing the pipeline run itself
- You need to evaluate a pre-existing assembly without running the full pipeline — use config_test_evaluation_only.yaml-style evaluation-only mode instead of full resource tuning
- The environment is not SLURM-based; use ./local_pipeline.sh with local resources rather than sbatch-based SLURM configuration

## Inputs

- config/config_template.yaml (Snakemake configfile template)
- Sample HiFi read FASTQ files (e.g. Mock community, Gut microbiome, Deadwood from doi:10.57745/2SA8AB)
- SLURM cluster environment with Snakemake >=8.28 and the SLURM executor plugin
- Optional external databases: Kraken 2 database and GTDB-Tk reference data

## Outputs

- outputs/<sample_name>/<assembler>/assembly.fasta per assembler
- outputs/<sample_name>/<assembler>/reads_on_contigs_mapping_evaluation/report.txt
- logs/<analysis_name>/<date_hour> log files with wall-clock timings
- execution_time_comparison.csv tabulating measured vs reference durations

## How to apply

Clone the Mapler repository and create the 'mapler' conda environment with 'bioconda::snakemake>=8.28', 'conda-forge::conda>=24.1.2', and the snakemake-executor-plugin-slurm plugin. Copy config/config_template.yaml as a per-dataset configfile, list sample read FASTQs and enable assemblers (metaMDBG, metaflye, hifiasm-meta) plus evaluation options (checkm, read_mapping_evaluation). Set resource parameters to the reported allocation of 48 threads and 200G memory per rule, since the README defines resource parameters as the knob for 'memory, threads, and execution time'. Optionally pre-build environments with 'snakemake --use-conda --conda-create-envs-only -c1' to speed the first execution, then launch with 'sbatch pipeline.sh <configfile>'. Collect wall-clock durations from logs/<analysis_name>/<date_hour> and output timestamps, and validate completion by checking outputs/<sample_name>/<assembler>/assembly.fasta and reads_on_contigs_mapping_evaluation/report.txt. The reference wall-clock times for three assemblers under 48 CPUs / 200G are 2 h 4 min (Mock community), 3 h 46 min (Gut microbiome), and 5 h 3 min (Deadwood); the small test dataset with 16 CPUs / 15G RAM should take ~15 minutes.

## Related tools

- **Snakemake** (Workflow engine whose configfile resource parameters set memory, threads, and execution time per rule) — https://github.com/Nimauric/Mapler
- **Conda** (Manages the mapler environment and pipeline dependencies, which can be pre-created with --conda-create-envs-only) — https://github.com/Nimauric/Mapler
- **snakemake-executor-plugin-slurm** (SLURM executor plugin enabling cluster submission via sbatch pipeline.sh <configfile>) — https://github.com/Nimauric/Mapler
- **Mapler** (Pipeline whose YAML config exposes functional and resource parameters that are tuned per dataset) — https://github.com/Nimauric/Mapler

## Examples

```
sbatch pipeline.sh config/config_mock.yaml
```

## Evaluation signals

- outputs/<sample_name>/<assembler>/assembly.fasta exists for every enabled assembler and sample, confirming the run completed
- outputs/<sample_name>/<assembler>/reads_on_contigs_mapping_evaluation/report.txt is present after each run
- Measured wall-clock times from logs are consistent with reference values (2 h 4 min Mock, 3 h 46 min Gut, 5 h 3 min Deadwood at 48 CPUs / 200G)
- Test dataset completes in ~15 minutes with 16 CPUs and 15G RAM (excluding dependency installation)
- Pre-built conda environments appear in .snakemake/conda when --conda-create-envs-only is used

## Limitations

- Wall-clock times are specific to the reported allocation (48 CPUs, 200G memory) and will differ on other hardware or allocations
- If taxonomic assignment is enabled, external Kraken 2 and GTDB-Tk databases must be provisioned and their paths given in the configfile (kraken2db, gtdbtk_database)
- A crashed run leaves the working directory locked, requiring 'snakemake --unlock --configfile <configfile> -np' before relaunch

## Evidence

- [readme] resource parameters allow users to configure memory, threads, and execution time: "functional parameters define values that can be adjusted for certain analyses, while resource parameters allow users to configure memory, threads, and execution time"
- [readme] SLURM executor plugin requirement: "The pipeline requires Snakemake and Conda. If running on a SLURM cluster, the Snakemake SLURM executor plugin is also needed."
- [readme] test dataset expected runtime: "with 16 CPUs and 15G of RAM, it should require around 15 minutes, not including dependency installation"
- [methods] SLURM executor plugin in methods: "the Snakemake SLURM executor plugin is also needed"
- [results] reported wall-clock durations: [evidence span withheld — anchor status: not_found]
