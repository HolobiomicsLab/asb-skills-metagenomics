---
name: sequence-alignment-benchmarking
description: Use when you need to determine the minimum RAM/CPU allocation for efficient
  alignment of a fixed-size sequence set (e.g. 50,000-sequence FASTA files) against
  a large local database (~6 GB nr Bacteria), or to find where added memory stops
  paying off.
when_to_use_negative:
- The input is a completed abundance/annotation summary table rather than raw sequence
  files to align
- The reference database already fits comfortably in available memory, so a memory-scaling
  benchmark would only measure constant overhead
- The goal is accuracy benchmarking of the aligner (e.g. fast vs. sensitive mode)
  rather than resource-scaling benchmarking
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0623
tools:
- name: DIAMOND
  role: Alignment engine whose annotation runtime is benchmarked; version 0.8.3 used,
    run in fast mode against a local DIAMOND-indexed database
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Standalone metatranscriptome pipeline providing the annotation workflow (DIAMOND
    against local databases) whose compute-resource requirements are characterized
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:257db1e68c747a8597e629f5b3d61b855f0719874e2131c5b1485a9a4772ae0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sequence-alignment-benchmarking

## Summary

A benchmarking skill for measuring how sequence-annotation runtime scales with allocated compute resources (RAM at fixed CPU), using SAMSA2's DIAMOND annotation against a local, DIAMOND-indexed reference database as the workload.

## When to use

Use when you need to determine the minimum RAM/CPU allocation for efficient alignment of a fixed-size sequence set (e.g. 50,000-sequence FASTA files) against a large local database (~6 GB nr Bacteria), or to find where added memory stops paying off.

## When NOT to use

- The input is a completed abundance/annotation summary table rather than raw sequence files to align
- The reference database already fits comfortably in available memory, so a memory-scaling benchmark would only measure constant overhead
- The goal is accuracy benchmarking of the aligner (e.g. fast vs. sensitive mode) rather than resource-scaling benchmarking

## Inputs

- Fixed-size test FASTA files (e.g. 50,000 sequences) extracted from a sample metatranscriptome
- Local DIAMOND-indexed reference database (~6 GB NCBI nr Bacteria, from full_database_download.bash)
- A compute environment permitting stepwise control of RAM allocation at fixed CPU/thread count

## Outputs

- Wall-clock runtime per RAM-allocation configuration
- Relative speedup table (runtime at lowest RAM / runtime at each allocation)
- Speedup-versus-RAM plot showing the saturation plateau
- Identified minimum effective resource configuration (~10 CPUs, 20 GB RAM)

## How to apply

Prepare fixed-size test FASTA files (e.g. 50,000 sequences) from the metatranscriptome reads so that file size is held constant across trials. Run DIAMOND (v0.8.3, used by SAMSA2) annotation of each test file against the local DIAMOND-indexed nr Bacteria database (~6 GB), sweeping RAM allocation stepwise while holding thread count fixed, and record wall-clock runtime per configuration. Compute relative speedup as runtime at the lowest RAM divided by runtime at each allocation, then plot speedup versus RAM to visualize the saturation point. Rationale: annotation speed improves only until the reference database is fully loaded into memory; beyond that plateau, extra RAM yields no gain. Validate that the curve plateaus once the ~6 GB database fits in memory and that the minimum effective configuration is approximately 10 CPUs / 20 GB RAM.

## Related tools

- **DIAMOND** (Alignment engine whose annotation runtime is benchmarked; version 0.8.3 used, run in fast mode against a local DIAMOND-indexed database) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Standalone metatranscriptome pipeline providing the annotation workflow (DIAMOND against local databases) whose compute-resource requirements are characterized) — https://github.com/transcript/samsa2

## Examples

```
bash example_DIAMOND_annotation_script.bash  # DIAMOND v0.8.3 --fast against the local DIAMOND-indexed nr Bacteria database, timing each RAM allocation
```

## Evaluation signals

- The speedup-versus-RAM curve plateaus at an allocation at or near the reference database size (~6 GB), confirming the database-fits-in-memory hypothesis
- The smallest tested configuration that achieves near-maximum speedup corresponds to approximately 10 CPUs and 20 GB RAM
- All trials use identical input size (50,000 sequences) and identical thread count, so runtime differences are attributable to RAM alone
- Each DIAMOND run completes successfully and produces per-read annotation output without memory errors at the tested allocations
- Runtimes are monotonically non-increasing (or constant) with increasing RAM, with no anomalous spikes

## Limitations

- Results are specific to DIAMOND v0.8.3 and the ~6 GB NCBI nr Bacteria database; larger databases shift the saturation point upward
- Switching DIAMOND from --fast to --sensitive mode may significantly increase memory requirements, invalidating the calibrated thresholds
- The benchmark reflects a fixed CPU setting; the 10 CPU / 20 GB minimum is only the efficient configuration for small (50K-sequence) files against this database
- SAMSA2's Python aggregation requires Python 2.7 (Python 3 support may contain uncaught errors), constraining where the pipeline can be run

## Evidence

- [abstract] SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [methods] use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed): "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [methods] DIAMOND, version 0.8.3: https://github.com/bbuchfink/diamond: "DIAMOND, version 0.8.3: https://github.com/bbuchfink/diamond"
- [readme] Note: DIAMOND is set by default to run in fast mode, where it prioritizes speed. If you are especially concerned about accuracy, you can change DIAMOND to sensitive mode: [evidence span withheld — anchor status: external_doc]
- [readme] Access the full databases by downloading them using the full_database_download.bash script: [evidence span withheld — anchor status: external_doc]
