---
name: nextflow-pipeline-execution
description: Use when shotgun virome FASTQ pairs must be processed end-to-end via
  'nextflow run deng-lab/viroprofiler' with a container profile and a '-c' custom
  config, or when per-process CPU/memory limits must be tuned.
when_to_use_negative:
- The analysis requires running isolated single tools outside a reproducible workflow
  rather than the full ViroProfiler chain
- The environment lacks Nextflow ≥21.10.3 or a container engine, so version-pinned
  docker/singularity execution is impossible
- Input is not shotgun viral metagenomic data (e.g., 16S amplicon data or samples
  with isolated viruses, which ViroProfiler is not designed for)
tools:
- name: ViroProfiler
  role: The Nextflow DSL2 pipeline being launched; executed via 'nextflow run denglab/viroprofiler'
  repo: https://github.com/deng-lab/viroprofiler
- name: Nextflow
  role: Workflow engine (DSL2, ≥21.10.3) that schedules processes and applies custom
    resource configs
- name: Docker
  role: Container profile used to pin tool versions for reproducible execution
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  - task_008
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:bfa5a7928a00ec2632eccdd007021488b744c852f23c11f74d98bd647a1b8539
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
  tools:
  - BACPHLIP
  - BBMap
  - Docker
  - FASTP
  - FastQC
  - MMseqs2
  - Nextflow
  - Replidec
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - iPhoP
  - metaSPAdes
  - vConTACT2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# nextflow-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Launch and parameterize the ViroProfiler viral metagenomics workflow through Nextflow DSL2 (≥21.10.3) with containerized dependencies (docker/singularity) and custom resource configs, so that trimming, assembly, viral-contig identification, taxonomy, and host/replication steps run reproducibly on local or cloud Linux environments.

## When to use

Use when shotgun virome FASTQ pairs must be processed end-to-end via 'nextflow run deng-lab/viroprofiler' with a container profile and a '-c' custom config, or when per-process CPU/memory limits must be tuned.

## When NOT to use

- The analysis requires running isolated single tools outside a reproducible workflow rather than the full ViroProfiler chain
- The environment lacks Nextflow ≥21.10.3 or a container engine, so version-pinned docker/singularity execution is impossible
- Input is not shotgun viral metagenomic data (e.g., 16S amplicon data or samples with isolated viruses, which ViroProfiler is not designed for)

## Inputs

- Paired-end shotgun virome FASTQ files (R1/R2) per sample
- Nextflow custom config file (CPU/memory per process) passed via '-c'
- Container engine (Docker or Singularity) and container images

## Outputs

- Executed ViroProfiler workflow with completed Nextflow processes (e.g., VIRALHOST_IPHOP, BACPHLIP, REPLIDEC)
- Cleaned FASTQ pairs, assemblies, viral contigs, taxonomy/annotation tables, and host/replication-cycle tables per sample
- Nextflow run report/log and error traces per process

## How to apply

Install Nextflow ≥21.10.3 and a container engine, then launch the pipeline as 'nextflow run denglab/viroprofiler -profile docker,test' to validate the installation against the bundled test dataset before committing real samples. For production runs, provide sample-level paired-end FASTQ inputs and pass an institutional resource config with '-c custom.config'; the ViroProfiler config assigns per-process resources such as 12 CPUs / 300 GB for VIRALHOST_IPHOP and 4 CPUs / 80 GB for BACPHLIP and REPLIDEC. Containerization via docker or singularity pins tool versions (FASTP, DECONTAM, metaSPAdes, VirSorter2, DeepVirFinder, VIBRANT, CheckV, iPhoP, MMseqs2) to guarantee computational reproducibility across local Linux machines and cloud environments. The same execution pattern is reused for benchmarking runs (e.g., Kraken2/Bracken comparisons on the 13-sample mock virome) and reanalysis of published SRA data (e.g., 20 UC fecal virome samples), keeping toolchain versions constant between compared pipelines.

## Related tools

- **ViroProfiler** (The Nextflow DSL2 pipeline being launched; executed via 'nextflow run denglab/viroprofiler') — https://github.com/deng-lab/viroprofiler
- **Nextflow** (Workflow engine (DSL2, ≥21.10.3) that schedules processes and applies custom resource configs)
- **Docker** (Container profile used to pin tool versions for reproducible execution)

## Examples

```
nextflow run denglab/viroprofiler -profile docker,test
```

## Evaluation signals

- All Nextflow processes (e.g., VIRALHOST_IPHOP, BACPHLIP, REPLIDEC) complete without error in the trace/report
- The docker,test profile runs successfully, confirming the installation, containers, and test dataset resolve correctly
- Downstream outputs exist per sample: cleaned FASTQ pair with equal R1/R2 read counts, plus populated per-contig host and replication-cycle tables
- Per-process resource assignments from the '-c' config (e.g., 12 CPUs/300 GB for VIRALHOST_IPHOP; 4 CPUs/80 GB for BACPHLIP) are honored in the run trace
- Re-running the pipeline reproduces identical results, verifying containerized reproducibility

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, not general metagenomes
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision
- Host/replication processes demand heavy resources (e.g., 300 GB for iPhoP), so machines below these limits may fail without a custom config

## Evidence

- [other] Test-profile invocation used to launch the QC module: [evidence span withheld — anchor status: not_found]
- [readme] Custom config pattern for per-process resources: [evidence span withheld — anchor status: not_found]
- [other] Custom config resource examples in ViroProfiler: [evidence span withheld — anchor status: not_found]
