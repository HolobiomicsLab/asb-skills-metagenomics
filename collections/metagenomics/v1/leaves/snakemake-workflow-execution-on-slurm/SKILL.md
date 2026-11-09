---
name: snakemake-workflow-execution-on-slurm
description: Use when reproducing or benchmarking the Mapler pipeline's wall-clock
  execution times for assembling and evaluating HiFi metagenomes (metaMDBG, metaflye,
  hifiasm-meta) under a fixed CPU/memory allocation on a SLURM cluster.
when_to_use_negative:
- The goal is only to install or smoke-test the pipeline — the test dataset (config/config_test.yaml)
  needs only 16 CPUs and 15G RAM for ~15 minutes, not the full 48 CPU / 200G benchmark
  allocation
- The input is already an assembled FASTA — use Mapler's evaluation-only mode (config_test_evaluation_only.yaml)
  rather than re-running assembly
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Mapler
  role: Snakemake pipeline being executed; assembles and evaluates HiFi long-read
    metagenomes
  repo: https://github.com/Nimauric/Mapler
- name: Snakemake
  role: Workflow engine; requires snakemake>=8.28 installed via Conda
- name: snakemake-executor-plugin-slurm
  role: SLURM executor plugin enabling 'sbatch pipeline.sh <configfile>' cluster execution
- name: Conda
  role: Manages the 'mapler' environment and per-rule pipeline dependencies under
    .snakemake/conda
- name: metaMDBG
  role: One of the three benchmarked HiFi metagenome assemblers run within the pipeline
  repo: https://github.com/GaetanBenoitDev/metaMDBG
- name: metaflye
  role: One of the three benchmarked HiFi metagenome assemblers run within the pipeline
  repo: https://github.com/mikolmogorov/Flye
- name: hifiasm-meta
  role: One of the three benchmarked HiFi metagenome assemblers run within the pipeline
  repo: https://github.com/xfengnefx/hifiasm-meta
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf334
    title: 'Mapler: a pipeline for assessing assembly quality in taxonomically rich
      metagenomes sequenced with HiFi reads'
  source_content_hash: sha256:8bf11c8b37ff0b19d68bfe50d5f5cc1f9b3c6a2915489fbfd8fbde45fd8cfd04
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
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# snakemake-workflow-execution-on-slurm

## Summary

Execute the Mapler Snakemake pipeline (metagenome assembly and evaluation for HiFi long reads) on a SLURM cluster using the Snakemake SLURM executor plugin, collecting per-assembler wall-clock durations from logs and output timestamps.

## When to use

Use when reproducing or benchmarking the Mapler pipeline's wall-clock execution times for assembling and evaluating HiFi metagenomes (metaMDBG, metaflye, hifiasm-meta) under a fixed CPU/memory allocation on a SLURM cluster.

## When NOT to use

- The goal is only to install or smoke-test the pipeline — the test dataset (config/config_test.yaml) needs only 16 CPUs and 15G RAM for ~15 minutes, not the full 48 CPU / 200G benchmark allocation
- The input is already an assembled FASTA — use Mapler's evaluation-only mode (config_test_evaluation_only.yaml) rather than re-running assembly

## Inputs

- Per-sample configfile (YAML) derived from config/config_template.yaml listing sample names, read FASTQ paths, assemblers, evaluation options, and resource parameters
- HiFi long-read FASTQ files per sample (Mock community, Gut microbiome, Deadwood from doi:10.57745/2SA8AB)
- Optional pre-downloaded Kraken 2 and GTDB-Tk databases if taxonomic assignment is enabled

## Outputs

- outputs/<sample_name>/<assembler>/assembly.fasta per assembler per sample
- outputs/<sample_name>/<assembler>/reads_on_contigs_mapping_evaluation/report.txt
- Execution logs in logs/<analysis_name>/<date_hour> with per-rule wall-clock durations
- execution_time_comparison.csv tabulating measured vs reference durations (2 h 4 min, 3 h 46 min, 5 h 3 min)

## How to apply

Clone the Mapler repository and create the 'mapler' conda environment with bioconda::snakemake>=8.28, conda-forge::conda>=24.1.2, and bioconda::snakemake-executor-plugin-slurm. Copy config/config_template.yaml as a per-dataset configfile listing sample reads (e.g. Mock community, Gut microbiome, Deadwood from doi:10.57745/2SA8AB) and enabling the assemblers metaMDBG, metaflye, and hifiasm-meta with evaluation options (checkm, read_mapping_evaluation). Set resource parameters to 48 threads and 200G memory per rule, optionally pre-build environments with 'snakemake --use-conda --conda-create-envs-only -c1', then launch each run with 'sbatch pipeline.sh <configfile>'. Measure wall-clock durations from logs in logs/<analysis_name>/<date_hour> and timestamps of outputs/<sample_name>/<assembler>/ artifacts, and tabulate them (e.g. Mock community 2 h 4 min, Gut microbiome 3 h 46 min, Deadwood 5 h 3 min) in execution_time_comparison.csv.

## Related tools

- **Mapler** (Snakemake pipeline being executed; assembles and evaluates HiFi long-read metagenomes) — https://github.com/Nimauric/Mapler
- **Snakemake** (Workflow engine; requires snakemake>=8.28 installed via Conda)
- **snakemake-executor-plugin-slurm** (SLURM executor plugin enabling 'sbatch pipeline.sh <configfile>' cluster execution)
- **Conda** (Manages the 'mapler' environment and per-rule pipeline dependencies under .snakemake/conda)
- **metaMDBG** (One of the three benchmarked HiFi metagenome assemblers run within the pipeline) — https://github.com/GaetanBenoitDev/metaMDBG
- **metaflye** (One of the three benchmarked HiFi metagenome assemblers run within the pipeline) — https://github.com/mikolmogorov/Flye
- **hifiasm-meta** (One of the three benchmarked HiFi metagenome assemblers run within the pipeline) — https://github.com/xfengnefx/hifiasm-meta

## Examples

```
git clone https://gitlab.inria.fr/mistic/mapler.git && cd mapler && conda create -n mapler -c bioconda -c conda-forge 'bioconda::snakemake>=8.28' 'conda-forge::conda>=24.1.2' bioconda::snakemake-executor-plugin-slurm && sbatch pipeline.sh config/config_deadwood.yaml
```

## Evaluation signals

- outputs/<sample_name>/<assembler>/assembly.fasta exists for every enabled assembler-sample combination, confirming each run completed
- outputs/<sample_name>/<assembler>/reads_on_contigs_mapping_evaluation/report.txt is present, confirming the read-mapping evaluation step finished
- Measured wall-clock times in execution_time_comparison.csv are consistent with the reference durations (2 h 4 min Mock community, 3 h 46 min Gut microbiome, 5 h 3 min Deadwood) under the 48 CPU / 200G allocation
- Conda environments resolve and pre-build successfully with 'snakemake --use-conda --conda-create-envs-only -c1' before the full run
- Log directories logs/<analysis_name>/<date_hour> contain complete per-rule timing records with no SLURM job failures

## Limitations

- Wall-clock times depend on the reported resource allocation (48 threads, 200G memory per rule); other allocations will not reproduce the reference durations
- The pipeline must be launched from within the pipeline directory; after a crash the working directory must be unlocked with 'snakemake --unlock --configfile <configfile> -np'
- Only part of the analysis runs on the bundled test dataset, and the test timing (~15 minutes with 16 CPUs / 15G RAM) excludes dependency installation time

## Evidence

- [methods] The pipeline requires Snakemake and Conda.: "The pipeline requires Snakemake and Conda."
- [methods] SLURM executor plugin required: "the Snakemake SLURM executor plugin is also needed"
- [readme] Environment setup command: "conda create -n mapler -c bioconda -c conda-forge 'bioconda::snakemake>=8.28' 'conda-forge::conda>=24.1.2' bioconda::snakemake-executor-plugin-slurm"
- [readme] Resource parameters: "resource parameters allow users to configure memory, threads, and execution time"
- [readme] Cluster launch command: "sbatch pipeline.sh <configfile> # for SLURM execution"
- [readme] Configfile recommendation: "It is recommended to copy `config/config_template.yaml` as a configfile, and to tweak it according to the analysis's needs."
