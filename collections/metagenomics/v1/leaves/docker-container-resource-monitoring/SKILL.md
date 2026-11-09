---
name: docker-container-resource-monitoring
description: Use when benchmarking the computational requirements (job runtime, CPU
  usage, peak memory) of tools executed inside Docker containers or through Nextflow
  pipelines, where measuring the orchestrator process would underestimate actual resource
  use.
when_to_use_negative:
- When measuring a non-containerized tool that runs as a single process — standard
  process monitoring suffices and container-level attribution is unnecessary
- When the goal is classification accuracy/sensitivity rather than computational cost
  — resource monitoring provides no signal about taxonomic assignment quality
- When the container environment itself restricts shared memory (e.g. missing --shm-size),
  as the measured memory profile would reflect container misconfiguration rather than
  the tool's true demand
tools:
- name: CAMITAX
  role: Pipeline-based classifier (v0.7.0, Nextflow v21.04.0) whose resource usage
    is measured through its Docker containers rather than the Nextflow main process
- name: Nextflow
  role: Orchestration engine (v21.04.0) for CAMITAX; explicitly NOT the process to
    monitor to avoid resource underestimation
- name: Docker
  role: Container runtime hosting the tool processes being monitored; kMetaShot container
    requires --shm-size=22g
- name: CMDbench
  role: Python package used to collect RAM, CPU%, and overall execution time for each
    tool
- name: kMetaShot
  role: Comparator tool (2.0 via bioconda) whose memory peak is dominated by loading
    the ~22 Gb HDF5 reference matrix
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparator tool (v1.0.2) monitored with standard process tracking for time
    and memory
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:89fb439ce08ac3b64abaf6ecb8a2e53e44fd724e47093e245c5be4db218a2f8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - CMDbench
  - Docker
  - GTDBtk
  - Nextflow
  - kMetaShot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# docker-container-resource-monitoring

## Summary

A method for measuring computational cost (runtime, CPU%, RAM) of containerized bioinformatics tools such as CAMITAX, whose Nextflow orchestration would otherwise cause resource measurements to be attributed to the main Nextflow process rather than the tool containers, leading to underestimation.

## When to use

Use when benchmarking the computational requirements (job runtime, CPU usage, peak memory) of tools executed inside Docker containers or through Nextflow pipelines, where measuring the orchestrator process would underestimate actual resource use.

## When NOT to use

- When measuring a non-containerized tool that runs as a single process — standard process monitoring suffices and container-level attribution is unnecessary
- When the goal is classification accuracy/sensitivity rather than computational cost — resource monitoring provides no signal about taxonomic assignment quality
- When the container environment itself restricts shared memory (e.g. missing --shm-size), as the measured memory profile would reflect container misconfiguration rather than the tool's true demand

## Inputs

- Benchmark MAG sets spanning low-to-high workload (e.g. CAMI II mock community multi-sample datasets with the lowest and largest numbers of MAGs)
- Containerized tool pipelines (e.g. CAMITAX v0.7.0 via Nextflow v21.04.0 with Docker)
- kMetaShot HDF5 reference matrix (~22 Gb on disk, requiring at least 32 Gb to load into memory)

## Outputs

- Per-tool job runtime, CPU%, and peak RAM measurements collected with CMDbench
- A combined comparison table saved as computational_requirements_table.csv

## How to apply

Run the pipeline-based tool (e.g. CAMITAX v0.7.0 via Nextflow v21.04.0) and attach resource monitoring to the underlying Docker containers instead of the main Nextflow process, which would otherwise report misleadingly low usage. Track per-container RAM, CPU%, and overall execution time using the CMDbench Python package, alongside comparable measurements for non-containerized tools (kMetaShot 2.0, GTDBtk v1.0.2) with standard process monitoring. For kMetaShot specifically, monitor the memory peak arising from loading the ~22 Gb HDF5 reference matrix into memory. Choose benchmark datasets spanning the workload range (fewest to most MAGs) so resource scaling is meaningful. For kMetaShot in Docker, allocate sufficient shared memory (--shm-size=22g) so high-parallelism runs are not artificially constrained. Assemble the collected values into a single comparison CSV (computational_requirements_table.csv) for the tools being compared.

## Related tools

- **CAMITAX** (Pipeline-based classifier (v0.7.0, Nextflow v21.04.0) whose resource usage is measured through its Docker containers rather than the Nextflow main process)
- **Nextflow** (Orchestration engine (v21.04.0) for CAMITAX; explicitly NOT the process to monitor to avoid resource underestimation)
- **Docker** (Container runtime hosting the tool processes being monitored; kMetaShot container requires --shm-size=22g)
- **CMDbench** (Python package used to collect RAM, CPU%, and overall execution time for each tool)
- **kMetaShot** (Comparator tool (2.0 via bioconda) whose memory peak is dominated by loading the ~22 Gb HDF5 reference matrix) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparator tool (v1.0.2) monitored with standard process tracking for time and memory)

## Examples

```
docker run -it quay.io/biocontainers/kmetashot kMetaShot_classifier_NV.py --help
```

## Evaluation signals

- kMetaShot reference-loading memory peak of at least 32 Gb is recorded, consistent with the paper's reported resource comparison
- CAMITAX's measured CPU usage reflects its Docker container processes, not the Nextflow main process (guard against underestimation)
- The assembled table records all three resources (RAM, CPU%, execution time) for kMetaShot, GTDBtk v1.0.2, and CAMITAX v0.7.0 across both low- and high-MAG-count benchmark datasets
- kMetaShot shows the shortest job runtimes and lowest CPU% usage across datasets, with memory use second only to CAMITAX
- Comparative CSV (computational_requirements_table.csv) exists and joins per-tool measurements collected with the same monitoring approach (CMDbench)

## Limitations

- kMetaShot's memory and CPU peaks correspond to loading the reference into memory, requiring at least 32 Gb, so measured peak memory largely reflects reference size rather than per-MAG workload
- Docker execution of kMetaShot requires --shm-size=22g; insufficient shared memory will distort or prevent measurement
- Monitoring only the Nextflow main process for CAMITAX would underestimate resource usage, so container-level attribution is mandatory for pipeline-based tools

## Evidence

- [discussion] the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb: "the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb"
- [methods] GTDBtk and CAMITAX versions with Nextflow version: "two state-of-art tools, namely, GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0)"
- [readme] kMetaShot docker shm-size requirement: "kMetaShot is also available as Docker container. It needs `--shm-size=22g` option to properly run as docker container."
- [readme] reference requires about 22Gb of storage: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
