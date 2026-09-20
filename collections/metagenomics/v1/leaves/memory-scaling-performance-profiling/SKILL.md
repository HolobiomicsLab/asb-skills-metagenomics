---
name: memory-scaling-performance-profiling
description: Decision support for sizing compute resources (CPUs/RAM) before running
  DIAMOND-based annotation of large FASTQ/FASTA metatranscriptome files against a
  local DIAMOND-indexed database such as NCBI nr Bacteria (~6 GB).
when_to_use_negative:
- The reference database is much smaller than the available RAM, so no memory-saturation
  regime will be observed
- The goal is annotation quality/accuracy benchmarking (e.g., DIAMOND --fast vs --sensitive
  mode comparison), not runtime scaling
- The pipeline step being profiled is not memory-bound (e.g., R aggregation or DESeq
  statistics on a personal computer)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: SAMSA2
  role: Pipeline whose DIAMOND annotation step is being benchmarked for RAM scaling
    on a supercomputing cluster
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner executing the annotation jobs whose runtime
    is measured under varying memory allocations
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:a85779ea63593122027bf909f13d44efca43b08836bf6b3aa3a4df9805e6c429
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# memory-scaling-performance-profiling

## Summary

A benchmarking method that profiles how DIAMOND annotation runtime in SAMSA2 scales with allocated RAM at fixed CPU, identifying the memory point where the reference database fully fits in memory and speedup saturates.

## When to use

Decision support for sizing compute resources (CPUs/RAM) before running DIAMOND-based annotation of large FASTQ/FASTA metatranscriptome files against a local DIAMOND-indexed database such as NCBI nr Bacteria (~6 GB).

## When NOT to use

- The reference database is much smaller than the available RAM, so no memory-saturation regime will be observed
- The goal is annotation quality/accuracy benchmarking (e.g., DIAMOND --fast vs --sensitive mode comparison), not runtime scaling
- The pipeline step being profiled is not memory-bound (e.g., R aggregation or DESeq statistics on a personal computer)

## Inputs

- Fixed-size test FASTA files (e.g., 50,000 sequences) derived from sample metatranscriptome reads
- Local DIAMOND-indexed reference database (e.g., ~6 GB NCBI nr Bacteria)

## Outputs

- Table of wall-clock runtimes per RAM allocation setting
- Relative speedup values (runtime at lowest RAM / runtime at each allocation)
- Speedup-versus-RAM curve showing the memory saturation point
- Minimum effective resource recommendation (~10 CPUs / 20 GB RAM)

## How to apply

Prepare fixed-size test FASTA files (e.g., 50,000 sequences) from the sample metatranscriptome reads so only the resource dimension varies. Run DIAMOND (v0.8.3) annotation of each test file against the local DIAMOND-indexed nr Bacteria database (~6 GB), sweeping RAM allocation stepwise while holding thread count fixed, and record wall-clock runtime for each configuration. Compute relative speedup as runtime at the lowest RAM allocation divided by runtime at each higher allocation. Plot speedup versus RAM to find the saturation point: increasing RAM only helps up to where the reference database is fully loaded into memory. From this curve, read off the minimum effective resource configuration, reported here as approximately 10 CPUs and 20 GB RAM.

## Related tools

- **SAMSA2** (Pipeline whose DIAMOND annotation step is being benchmarked for RAM scaling on a supercomputing cluster) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner executing the annotation jobs whose runtime is measured under varying memory allocations) — https://github.com/bbuchfink/diamond

## Examples

```
bash example_DIAMOND_annotation_script.bash  # run DIAMOND annotation against the local nr Bacteria db, varying RAM allocation per SLURM submission and recording runtime
```

## Evaluation signals

- The speedup-versus-RAM curve plateaus once RAM exceeds the size of the DIAMOND-indexed database (~6 GB), indicating the database is fully memory-resident
- Runtimes are strictly non-increasing with additional RAM below the saturation point
- The measured minimum effective configuration agrees with the reported ~10 CPUs / 20 GB RAM for efficient processing of small files against the ~6 GB database
- All runs use identical fixed-size input (50,000 sequences) and fixed thread count, so speedup differences are attributable to RAM alone
- Re-running a configuration reproduces the recorded wall-clock runtime within expected cluster variability

## Limitations

- The minimum of 10 CPUs/20 GB RAM applies to processing even small 50,000-sequence files against the ~6 GB NCBI nr Bacteria database; larger databases or inputs will shift the saturation point
- Increasing RAM beyond the point where the database fully fits in memory yields no further speedup
- DIAMOND was benchmarked at version 0.8.3 in default --fast mode; using --sensitive mode 'may significantly increase memory requirements', changing the scaling profile

## Evidence

- [full_text] Higher RAM allocation speeds DIAMOND annotation only up to the point where the reference database is fully in memory; a minimum of 10 CPUs/20 GB RAM is needed to efficiently process even small files against the ~6 GB NCBI nr Bacteria database.: [evidence span withheld — anchor status: not_found]
- [readme] DIAMOND is set by default to run in fast mode, where it prioritizes speed.: [evidence span withheld — anchor status: external_doc]
- [methods] DIAMOND is an accelerated BLAST-like sequence aligner used for annotation against a local, DIAMOND-indexed database.: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [abstract] SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [readme] Switching DIAMOND to sensitive mode may significantly increase memory requirements.: [evidence span withheld — anchor status: external_doc]
