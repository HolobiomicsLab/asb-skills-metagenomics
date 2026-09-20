---
name: computational-resource-profiling
description: Use when comparing the CPU time, runtime, and peak memory of genome taxonomy
  classifiers on the same MAG sets, or when planning hardware needs such as the ~22Gb
  HDF5 kMetaShot reference load.
when_to_use_negative:
- The goal is classification accuracy (sensitivity, balanced accuracy, FPR) rather
  than computational cost — this skill measures RAM/CPU/time only, not correctness
  of taxonomy assignments
- Input is raw shotgun reads rather than binned MAGs — the classifiers compared here
  operate directly on binning output FASTA files
- Only one tool is being profiled with no comparison target — the skill's rationale
  is the cross-tool comparison table
tools:
- name: kMetaShot
  role: Classifier under test whose runtime, CPU%, and reference-loading memory peak
    (~22Gb HDF5, ≥32Gb RAM) are measured
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparison tool (v1.0.2) run on the same MAG bins with standard process monitoring
    of time and memory
- name: CAMITAX
  role: Comparison tool (v0.7.0) run through Nextflow v21.04.0; resources measured
    via its Docker containers
- name: Nextflow
  role: Execution framework (v21.04.0) for CAMITAX; measurement must target the tool's
    containers, not the main Nextflow process
- name: Docker
  role: Container runtime for CAMITAX and kMetaShot (quay.io/biocontainers/kmetashot,
    needing --shm-size=22g)
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
  source_content_hash: sha256:a2a39aa597b0143069a91efa8ce68092fbcc1d1b312b170378ca323c3ddc2116
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
yes_rate: 0.5
n_claims: 10
---

# computational-resource-profiling

## Summary

Benchmarks the computational footprint (job runtime, CPU%, peak RAM) of MAG taxonomy classifiers such as kMetaShot, GTDBtk, and CAMITAX across benchmark datasets, assembling measurements into a comparative CSV table. It exists to quantify whether an alignment-free classifier like kMetaShot is cheaper to run than marker-gene/alignment-based tools.

## When to use

Use when comparing the CPU time, runtime, and peak memory of genome taxonomy classifiers on the same MAG sets, or when planning hardware needs such as the ~22Gb HDF5 kMetaShot reference load.

## When NOT to use

- The goal is classification accuracy (sensitivity, balanced accuracy, FPR) rather than computational cost — this skill measures RAM/CPU/time only, not correctness of taxonomy assignments
- Input is raw shotgun reads rather than binned MAGs — the classifiers compared here operate directly on binning output FASTA files
- Only one tool is being profiled with no comparison target — the skill's rationale is the cross-tool comparison table

## Inputs

- Benchmark MAG bins (FASTA) from datasets spanning the workload range, e.g. CAMI II multi-sample mock community datasets
- kMetaShot HDF5 reference (~22Gb storage matrix, e.g. kMetaShot_bacteria_archaea_2025-05-22.h5)
- GTDBtk v1.0.2 installation and CAMITAX v0.7.0 Nextflow v21.04.0 workflow with Docker containers
- CMDbench Python package for resource monitoring

## Outputs

- computational_requirements_table.csv with RAM, CPU%, and execution time per tool per dataset
- Per-tool resource profiles showing kMetaShot reference-loading memory peak (at least 32Gb)

## How to apply

Select benchmark MAG datasets spanning the workload range (e.g., the CAMI II datasets with the lowest and largest numbers of MAGs). Run kMetaShot on each dataset while tracking the memory peak caused by loading the ~22Gb HDF5 storage matrix reference (which requires at least 32Gb RAM); note that parallelism (-p processes) drives RAM usage. Run GTDBtk (v1.0.2) on the same bins with standard process monitoring, and run CAMITAX (v0.7.0) through Nextflow v21.04.0, measuring resources via its Docker containers rather than the main Nextflow process to avoid underestimation. Collect RAM, CPU%, and overall execution time for every tool with the CMDbench Python package, then assemble the measured values into a single CSV (computational_requirements_table.csv).

## Related tools

- **kMetaShot** (Classifier under test whose runtime, CPU%, and reference-loading memory peak (~22Gb HDF5, ≥32Gb RAM) are measured) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparison tool (v1.0.2) run on the same MAG bins with standard process monitoring of time and memory)
- **CAMITAX** (Comparison tool (v0.7.0) run through Nextflow v21.04.0; resources measured via its Docker containers)
- **Nextflow** (Execution framework (v21.04.0) for CAMITAX; measurement must target the tool's containers, not the main Nextflow process)
- **Docker** (Container runtime for CAMITAX and kMetaShot (quay.io/biocontainers/kmetashot, needing --shm-size=22g))
- **CMDbench** (Python package used to collect RAM, CPU%, and overall execution time for each tool)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5 -p 10 -o output_dir -a 0.1
```

## Evaluation signals

- computational_requirements_table.csv exists and contains RAM, CPU%, and execution time rows for kMetaShot, GTDBtk, and CAMITAX across the selected datasets
- The table records the kMetaShot memory peak attributable to reference loading of at least 32Gb
- kMetaShot shows the shortest job runtimes and lowest CPU% usage across datasets, with memory use second only to CAMITAX
- CAMITAX resource numbers were captured from its Docker containers, not the Nextflow master process (guarding against underestimation)
- Measured kMetaShot peak RAM increases with the -p parallelism level, consistent with 'high parallelism <==> high RAM usage'

## Limitations

- kMetaShot's memory and CPU peak corresponds to loading the HDF5 reference into memory, requiring at least 32Gb; the Docker image additionally needs --shm-size=22g
- Measuring CAMITAX through the main Nextflow process underestimates its true resource usage; container-level measurement is required
- High kMetaShot parallelism (-p) directly trades speed for RAM, so resource numbers depend on the chosen process count

## Evidence

- [discussion] the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb.: "the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb."
- [readme] kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage.: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
- [readme] Warning: high parallelism <==> high RAM usage: "Warning: high parallelism <==> high RAM usage"
- [methods] GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0): "two state-of-art tools, namely, GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0)"
- [readme] It needs --shm-size=22g option to properly run as docker container.: "It needs `--shm-size=22g` option to properly run as docker container."
