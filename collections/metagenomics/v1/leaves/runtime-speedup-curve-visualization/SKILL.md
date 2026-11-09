---
name: runtime-speedup-curve-visualization
description: Use when you need to determine the minimum CPU/RAM allocation at which
  an aligner or pipeline step reaches peak efficiency, e.g. before sizing cluster
  jobs for SAMSA2 DIAMOND annotation runs.
when_to_use_negative:
- The goal is annotating a full production sample rather than characterizing resource
  scaling — run the standard SAMSA2 master_script pipeline instead of a benchmark
  sweep.
- The reference database is small enough to trivially fit in the target node's memory,
  making a saturation benchmark unnecessary.
- The tool is not I/O- or memory-bound (e.g. single-threaded personal-computer aggregation
  scripts like DIAMOND_analysis_counter.py), where RAM-scaling curves would be uninformative.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DIAMOND
  role: The BLAST-like aligner whose annotation runtime is benchmarked across RAM
    allocations, run against a local DIAMOND-indexed nr Bacteria database (version
    0.8.3).
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline providing the DIAMOND annotation step, example annotation script,
    and sample metatranscriptome data used to construct the benchmark test files.
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:dfad79f30a31d0b07034490f264da53105a53d59fb4c36d71f523be84553530a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# runtime-speedup-curve-visualization

## Summary

Benchmark wall-clock runtime of a tool (here, DIAMOND v0.8.3 annotation within SAMSA2) as a compute resource is swept stepwise, then compute and plot relative speedup to identify the resource saturation point. This reveals when adding resources (e.g. RAM to hold the ~6 GB NCBI nr Bacteria database in memory) stops yielding further gains.

## When to use

Use when you need to determine the minimum CPU/RAM allocation at which an aligner or pipeline step reaches peak efficiency, e.g. before sizing cluster jobs for SAMSA2 DIAMOND annotation runs.

## When NOT to use

- The goal is annotating a full production sample rather than characterizing resource scaling — run the standard SAMSA2 master_script pipeline instead of a benchmark sweep.
- The reference database is small enough to trivially fit in the target node's memory, making a saturation benchmark unnecessary.
- The tool is not I/O- or memory-bound (e.g. single-threaded personal-computer aggregation scripts like DIAMOND_analysis_counter.py), where RAM-scaling curves would be uninformative.

## Inputs

- Fixed 50,000-sequence test FASTA files derived from metatranscriptome reads
- Local DIAMOND-indexed reference database (e.g. ~6 GB NCBI nr Bacteria / RefSeq bacteria DIAMOND database)
- Compute resource configurations (stepwise RAM allocations at fixed CPU thread count)

## Outputs

- Wall-clock runtime table for each RAM/CPU configuration
- Relative speedup values (runtime at lowest RAM / runtime at each allocation)
- Speedup-vs-RAM plot showing the memory saturation plateau
- Identified minimum effective resource configuration (~10 CPUs / 20 GB RAM)

## How to apply

Prepare fixed-size test inputs (e.g. 50,000-sequence FASTA files extracted from a metatranscriptome sample) so file size does not confound timing. Run DIAMOND annotation of each test file against the local DIAMOND-indexed reference database (e.g. NCBI nr Bacteria, ~6 GB), sweeping one resource axis (RAM) stepwise while holding the other (thread count) fixed, and record wall-clock runtime per configuration. Compute relative speedup as runtime at the lowest allocation divided by runtime at each allocation. Plot speedup versus RAM allocation; the curve plateaus once the ~6 GB reference database fits fully in memory, indicating allocations beyond that point are wasted. Validate that the minimum effective configuration (reported as approximately 10 CPUs / 20 GB RAM for this database) reproduces plateau-level performance.

## Related tools

- **DIAMOND** (The BLAST-like aligner whose annotation runtime is benchmarked across RAM allocations, run against a local DIAMOND-indexed nr Bacteria database (version 0.8.3).) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline providing the DIAMOND annotation step, example annotation script, and sample metatranscriptome data used to construct the benchmark test files.) — https://github.com/transcript/samsa2

## Examples

```
bash example_DIAMOND_annotation_script.bash  # annotate fixed 50K-seq FASTA vs local DIAMOND-indexed nr Bacteria DB at each RAM allocation, timing each run
```

## Evaluation signals

- Speedup-vs-RAM curve plateaus once allocated RAM exceeds the ~6 GB size of the DIAMOND-indexed nr Bacteria database.
- Runtimes for identical 50,000-sequence test files are reproducible at each configuration, with file size held constant across the sweep.
- Minimum effective configuration is approximately 10 CPUs / 20 GB RAM; configurations below this show materially longer wall-clock times.
- Annotations produced at different RAM allocations map the same reads to the same database hits, confirming resource changes affect speed but not results.
- Thread count is verifiably constant across all benchmark runs so speedup differences are attributable to memory alone.

## Limitations

- DIAMOND is set by default to run in fast mode for speed; switching to --sensitive mode (by editing lines 162 and 199 of master_script) may significantly increase memory requirements, shifting the saturation point.
- The benchmark reflects SAMSA2's DIAMOND v0.8.3 and the ~6 GB nr Bacteria database; other database sizes or DIAMOND versions will shift the plateau.
- SAMSA2's aggregation scripts require Python 2.7 (Python 3 support incomplete) and the pipeline was only tested on Linux, constraining where benchmark infrastructure can run.

## Evidence

- [methods] DIAMOND local database annotation requirement: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [abstract] SAMSA2 speed rationale: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [readme] DIAMOND default fast mode and memory caveat: [evidence span withheld — anchor status: external_doc]
- [readme] Sensitive mode memory warning: [evidence span withheld — anchor status: external_doc]
