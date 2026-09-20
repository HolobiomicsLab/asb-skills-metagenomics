---
name: cluster-resource-allocation
description: Decision support for allocating cluster CPUs and per-task memory before
  running SAMSA2/DIAMOND annotation jobs, or when estimating annotation wall-clock
  time from input FASTQ size.
when_to_use_negative:
- Input is already annotated or the task is downstream aggregation/DESeq statistics
  rather than scheduling the CPU-intensive annotation step
- Running on a web service such as MG-RAST, where compute resources and parameters
  are outside the user's control
- Single small samples where estimating runtime from file size adds no value
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Pipeline whose CPU-intensive annotation stage is being scheduled and benchmarked
    on the cluster
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner whose CPU/RAM footprint (fast vs. sensitive
    mode) drives the resource allocation
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:e3359b724904d027078c6eadec1341e172da6f0165d2b813264aff856741206a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cluster-resource-allocation

## Summary

Guidance for sizing and holding constant compute resources (CPUs and RAM per task) when running SAMSA2's DIAMOND annotation on an HPC cluster, so that runtime scales predictably and linearly with input metatranscriptome size.

## When to use

Decision support for allocating cluster CPUs and per-task memory before running SAMSA2/DIAMOND annotation jobs, or when estimating annotation wall-clock time from input FASTQ size.

## When NOT to use

- Input is already annotated or the task is downstream aggregation/DESeq statistics rather than scheduling the CPU-intensive annotation step
- Running on a web service such as MG-RAST, where compute resources and parameters are outside the user's control
- Single small samples where estimating runtime from file size adds no value

## Inputs

- SAMSA2 packaged sample metatranscriptome read files spanning increasing sizes (FASTQ)
- Local, DIAMOND-indexed reference protein database (e.g., NCBI RefSeq Bacteria, SEED Subsystems)

## Outputs

- Wall-clock annotation time per input size (annotation_time_scaling.csv)
- Runtime growth rate between successive input-size points confirming approximately linear scaling

## How to apply

Run SAMSA2's DIAMOND annotation (DIAMOND version 0.8.3, as distributed with SAMSA2) against a local, DIAMOND-indexed reference database while holding compute resources constant at 30 CPUs and 2 GB RAM per run; the article reports annotation time increases approximately linearly with input sequence count, so runtime can be estimated from file size. Format the run as an HPC job — the SAMSA2 master script is written for a SLURM scheduler, though this can be changed or removed — and rely on the checkpoint file it writes so finished CPU-intensive steps (PEAR, Trimmomatic, SortMeRNA) are skipped on reruns. Keep DIAMOND in its default --fast mode, which prioritizes speed; switch to --sensitive only if accuracy matters, since this may significantly increase memory requirements. Record wall-clock time and input size (reads or bases) for each run and compute the runtime growth rate between successive size points to confirm approximately linear scaling.

## Related tools

- **SAMSA2** (Pipeline whose CPU-intensive annotation stage is being scheduled and benchmarked on the cluster) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner whose CPU/RAM footprint (fast vs. sensitive mode) drives the resource allocation) — https://github.com/bbuchfink/diamond

## Examples

```
sbatch --cpus-per-task=30 --mem-per-cpu=2G master_script.bash
```

## Evaluation signals

- annotation_time_scaling.csv exists and contains a runtime entry for every input size tested
- Successive runtime increments remain approximately proportional to corresponding input-size increments (linear scaling)
- Compute resources were identical across all runs (30 CPUs, 2 GB RAM per run) so timings are comparable
- Checkpoint file shows completed CPU-intensive steps (PEAR, Trimmomatic, SortMeRNA) were skipped on rerun rather than re-executed

## Limitations

- DIAMOND is pinned to version 0.8.3 as distributed with SAMSA2; other versions may change the runtime profile
- Running DIAMOND in --sensitive mode may significantly increase memory requirements versus the default --fast mode
- SAMSA2 requires Python 2.7 for aggregation scripts (tested on Linux only); the README warns some Python 3 errors may not yet be caught
- Linear runtime scaling is established only over the range of packaged sample file sizes tested

## Evidence

- [abstract] SAMSA2 is an upgrade to the original Simple Annotation of Metatranscriptomes by Sequence Analysis (SAMSA) pipeline that has been redesigned for standalone use on a supercomputing cluster.: "SAMSA2 is an upgrade to the original Simple Annotation of Metatranscriptomes by Sequence Analysis (SAMSA) pipeline that has been redesigned for standalone use on a supercomputing cluster."
- [readme] SLURM formatting and checkpoint skipping of CPU-intensive steps: "The script is formated to be run on a HPC using a SLURM job scheduler, but this can be easily changed / removed."
- [readme] Fast vs. sensitive mode memory tradeoff: [evidence span withheld — anchor status: external_doc]
- [readme] Rationale for DIAMOND speed on large inputs: "At a standard rate of 10 annotations per second, a standard BLAST approach would take several months to finish - just for a single file!"
