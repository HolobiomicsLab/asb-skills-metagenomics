---
name: benchmark-results-tabulation
description: Use when comparing runtime, CPU usage, and peak memory of MAG taxonomy
  classifiers across the same benchmark datasets, especially when one tool's cost
  is dominated by reference loading (e.g. kMetaShot's ~22Gb HDF5 matrix requiring
  ≥32Gb RAM).
when_to_use_negative:
- When the goal is classification accuracy (sensitivity, false positive rate, balanced
  accuracy) rather than computational cost — use accuracy benchmark workflows instead
- When measuring a tool whose resource use cannot be attributed to a single container
  or process (e.g. orchestration wrappers whose main process underestimates true consumption)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: kMetaShot
  role: Classifier under benchmark; its ~22Gb HDF5 reference loading dominates its
    memory peak (≥32Gb)
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparison tool benchmarked at v1.0.2 with standard process monitoring
- name: CAMITAX
  role: Comparison tool run via Nextflow (v21.04.0); resources measured through its
    Docker containers
- name: Nextflow
  role: Workflow engine (v21.04.0) used to execute CAMITAX during benchmarking
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
  source_content_hash: sha256:7011e2de5bbd5ffb6a08723c477b68eec711732a8ea2bbb16f6168a4cecc5e5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - CMDbench
  - GTDBtk
  - Nextflow
  - kMetaShot
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# benchmark-results-tabulation

## Summary

Tabulates computational resource measurements (RAM, CPU%, execution time) for kMetaShot, GTDBtk, and CAMITAX across benchmark MAG datasets into a single comparison table (computational_requirements_table.csv), enabling reproducible tool-efficiency comparison.

## When to use

Use when comparing runtime, CPU usage, and peak memory of MAG taxonomy classifiers across the same benchmark datasets, especially when one tool's cost is dominated by reference loading (e.g. kMetaShot's ~22Gb HDF5 matrix requiring ≥32Gb RAM).

## When NOT to use

- When the goal is classification accuracy (sensitivity, false positive rate, balanced accuracy) rather than computational cost — use accuracy benchmark workflows instead
- When measuring a tool whose resource use cannot be attributed to a single container or process (e.g. orchestration wrappers whose main process underestimates true consumption)

## Inputs

- Benchmark MAG datasets (e.g. CAMI II multi-sample mock community datasets, binned with MetaBAT2 v2:2.15 with --maxP 99 --minS 98 -m 1,500)
- kMetaShot HDF5 reference matrix (~22Gb, e.g. kMetaShot_bacteria_archaea_2025-05-22.h5)
- Process/resource monitoring traces from CMDbench for each tool run
- CAMITAX Docker container resource logs from Nextflow (v21.04.0) execution

## Outputs

- computational_requirements_table.csv with RAM, CPU%, and overall execution time for kMetaShot, GTDBtk, and CAMITAX per dataset

## How to apply

Select benchmark MAG datasets spanning the workload range (lowest and largest number of MAGs), then run each classifier while tracking resources with the CMDbench Python package. Run kMetaShot noting the memory peak from loading the ~22Gb HDF5 storage matrix reference; run GTDBtk (v1.0.2) with standard process monitoring; run CAMITAX (v0.7.0) through Nextflow (v21.04.0) and measure resources via its Docker containers rather than the main Nextflow process to avoid underestimation. Assemble the measured RAM, CPU%, and execution-time values into a single CSV table. Validate that the table records kMetaShot's reference-loading memory peak of at least 32Gb and that CAMITAX shows the lowest CPU usage among the compared tools.

## Related tools

- **kMetaShot** (Classifier under benchmark; its ~22Gb HDF5 reference loading dominates its memory peak (≥32Gb)) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparison tool benchmarked at v1.0.2 with standard process monitoring)
- **CAMITAX** (Comparison tool run via Nextflow (v21.04.0); resources measured through its Docker containers)
- **Nextflow** (Workflow engine (v21.04.0) used to execute CAMITAX during benchmarking)
- **CMDbench** (Python package used to collect RAM, CPU%, and overall execution time for each tool)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- The table records kMetaShot's memory peak of at least 32 Gb arising from reference loading
- CAMITAX exhibits the lowest CPU% usage among the three compared tools, consistent with the paper's reported comparison
- kMetaShot shows the shortest job runtimes and shortest loading time across the benchmark datasets
- Each dataset row contains RAM, CPU%, and execution-time values for all three tools, sourced from CMDbench measurements

## Limitations

- kMetaShot's reference loading requires at least 32Gb RAM; high process parallelism (-p) further increases RAM usage, so benchmarks need adequate memory
- Measuring CAMITAX via the main Nextflow process would underestimate resources; Docker container-level monitoring is required
- The kMetaShot Docker container needs --shm-size=22g to run properly, which constrains containerized benchmarking setups

## Evidence

- [discussion] the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb.: "the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb."
- [methods] GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0): "two state-of-art tools, namely, GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0)"
- [readme] kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage.: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
- [readme] Warning: high parallelism <==> high RAM usage: "Warning: high parallelism <==> high RAM usage"
