---
name: runtime-and-memory-measurement
description: Use when you must compare CPU time, memory peaks, and job runtimes of
  multiple MAG taxonomic classifiers on the same benchmark MAG sets and need a reproducible
  resource comparison table.
when_to_use_negative:
- The goal is to compare classification accuracy/sensitivity rather than computational
  cost — the accuracy benchmarking uses different inputs and metrics (sensitivity,
  balanced accuracy, FPR).
- Only one tool and one workload point are being profiled — the comparison protocol
  requires at least two datasets spanning the MAG-count range and multiple tools.
- Containerized tools would be monitored via the orchestrator process (e.g., the main
  Nextflow process), which the paper explicitly flags as underestimating resources.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: kMetaShot
  role: Classifier under benchmark; its runtime and the memory peak from loading the
    ~22Gb HDF5 reference are measured
  repo: github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparator tool (v1.0.2) run on the same datasets with standard process monitoring
    of time and memory
- name: CAMITAX
  role: Comparator tool (v0.7.0) run via Nextflow v21.04.0 with resources measured
    through its Docker containers
- name: Nextflow
  role: Workflow engine (v21.04.0) executing CAMITAX during the benchmark
- name: Docker
  role: Container runtime whose processes are monitored to capture CAMITAX's true
    resource usage
- name: CMDbench
  role: Python package used to collect RAM, CPU%, and overall execution time for each
    tool
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:870eb0fc7eebaad2e95490eae64348fc10525a5742f88f30043ec9dd074acde2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMITAX
  - CMDbench
  - Docker
  - GTDBtk
  - Nextflow
  - kMetaShot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# runtime-and-memory-measurement

## Summary

Benchmark protocol for comparing computational efficiency (job runtime, CPU%, and peak RAM) of MAG taxonomy classifiers such as kMetaShot, GTDBtk, and CAMITAX, measured with CMDbench and assembled into a single CSV resource table.

## When to use

Use when you must compare CPU time, memory peaks, and job runtimes of multiple MAG taxonomic classifiers on the same benchmark MAG sets and need a reproducible resource comparison table.

## When NOT to use

- The goal is to compare classification accuracy/sensitivity rather than computational cost — the accuracy benchmarking uses different inputs and metrics (sensitivity, balanced accuracy, FPR).
- Only one tool and one workload point are being profiled — the comparison protocol requires at least two datasets spanning the MAG-count range and multiple tools.
- Containerized tools would be monitored via the orchestrator process (e.g., the main Nextflow process), which the paper explicitly flags as underestimating resources.

## Inputs

- Benchmark MAG sets (e.g., in silico CAMI II multi-sample mock community datasets) spanning lowest and largest bin counts
- kMetaShot HDF5 reference matrix (~22Gb, e.g., kMetaShot_bacteria_archaea_2025-05-22.h5)
- GTDBtk v1.0.2 installation
- CAMITAX v0.7.0 Nextflow (v21.04.0) pipeline with Docker containers
- CMDbench Python package for process monitoring

## Outputs

- computational_requirements_table.csv with RAM, CPU%, and overall execution time per tool per dataset
- Per-tool runtime, CPU usage, and memory peak measurements, including the kMetaShot reference-loading memory peak

## How to apply

Select two benchmark datasets containing the lowest and largest numbers of MAGs so the workload range is spanned. Run kMetaShot on both datasets, separately tracking the memory peak caused by loading the ~22Gb HDF5 reference storage matrix (which requires at least 32 Gb in memory). Run GTDBtk (v1.0.2) with standard process monitoring on the same datasets. Run CAMITAX (v0.7.0) through Nextflow (v21.04.0) but measure resources via its Docker containers rather than the main Nextflow process, since the latter underestimates true consumption. Collect RAM, CPU%, and overall execution time for each tool with the CMDbench Python package and assemble the values into computational_requirements_table.csv.

## Related tools

- **kMetaShot** (Classifier under benchmark; its runtime and the memory peak from loading the ~22Gb HDF5 reference are measured) — github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparator tool (v1.0.2) run on the same datasets with standard process monitoring of time and memory)
- **CAMITAX** (Comparator tool (v0.7.0) run via Nextflow v21.04.0 with resources measured through its Docker containers)
- **Nextflow** (Workflow engine (v21.04.0) executing CAMITAX during the benchmark)
- **Docker** (Container runtime whose processes are monitored to capture CAMITAX's true resource usage)
- **CMDbench** (Python package used to collect RAM, CPU%, and overall execution time for each tool)

## Evaluation signals

- The CSV table records a kMetaShot reference-loading memory peak of at least 32 Gb, consistent with the paper's discussion.
- CAMITAX shows the lowest CPU usage among the three compared tools, matching the paper's reported resource comparison.
- kMetaShot shows the shortest job runtimes and lowest CPU% usage across datasets, with memory second only to CAMITAX.
- Measurements for CAMITAX were captured from its Docker containers, not the main Nextflow process.
- The two selected datasets correspond to the lowest and largest MAG counts, confirming the workload range is spanned.

## Limitations

- The kMetaShot memory and CPU peak corresponds to loading the reference storage matrix, requiring at least 32Gb, so machines with less RAM cannot run this benchmark as designed.
- kMetaShot reference requires about 22Gb of storage, and high process parallelism increases RAM usage (the README warns 'high parallelism <==> high RAM usage').
- As a Docker container, kMetaShot needs the --shm-size=22g option to run properly.
- Monitoring CAMITAX via the main Nextflow process underestimates its true resource consumption.

## Evidence

- [discussion] reference loading memory peak: "the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb."
- [intro] less expensive than benchmarked tools: "we also demonstrated that kMetaShot is computationally less expensive compared to the benchmarked tools, with very user-friendly installation and usage procedures"
- [methods] compared tool versions: "two state-of-art tools, namely, GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0)"
- [readme] reference storage size: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
- [readme] parallelism RAM warning: "Number of child processes for a Multiprocess parallelism. Warning: high parallelism <==> high RAM usage"
