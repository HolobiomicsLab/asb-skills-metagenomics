---
name: diamond-aligner-benchmarking
description: Use when you need to predict or budget DIAMOND annotation compute time
  for metatranscriptome samples by measuring how SAMSA2's DIAMOND runtime scales with
  the number of input sequences under a fixed CPU/memory allocation.
when_to_use_negative:
- The goal is annotation accuracy or genus-level classification quality rather than
  runtime scaling — use SAMSA2's accuracy comparison workflow instead
- Input reads have not passed SAMSA2 preprocessing (PEAR merging, Trimmomatic cleaning,
  SortMeRNA ribosomal filtering), so annotated size would not reflect the mRNA fraction
  being benchmarked
- Compute resources vary between runs, since non-constant CPU/RAM allocation would
  confound the linear scaling measurement
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: DIAMOND
  role: Accelerated BLAST-like aligner whose annotation runtime is benchmarked against
    input size; run in fast mode against a local DIAMOND-indexed database
  repo: https://github.com/bbuchfink/diamond
- name: SAMSA2
  role: Pipeline providing the DIAMOND annotation step, packaged sample metatranscriptome
    files, and database download scripts used in the benchmark
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:f3f08ac946fbf8b96cf46a101a2001f885ca9cf761cae2889fc24065b40f5b35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# diamond-aligner-benchmarking

## Summary

Benchmarks DIAMOND-based annotation runtime scaling in SAMSA2 by running DIAMOND (version 0.8.3) over metatranscriptome input files of increasing sequence count under fixed compute resources, then verifying that wall-clock annotation time grows approximately linearly with input size so runtime can be estimated from file size.

## When to use

Use when you need to predict or budget DIAMOND annotation compute time for metatranscriptome samples by measuring how SAMSA2's DIAMOND runtime scales with the number of input sequences under a fixed CPU/memory allocation.

## When NOT to use

- The goal is annotation accuracy or genus-level classification quality rather than runtime scaling — use SAMSA2's accuracy comparison workflow instead
- Input reads have not passed SAMSA2 preprocessing (PEAR merging, Trimmomatic cleaning, SortMeRNA ribosomal filtering), so annotated size would not reflect the mRNA fraction being benchmarked
- Compute resources vary between runs, since non-constant CPU/RAM allocation would confound the linear scaling measurement

## Inputs

- DIAMOND-indexed local NCBI nr Bacteria protein reference database (downloaded via full_database_download.bash)
- SAMSA2-packaged sample metatranscriptome FASTQ read files of increasing sizes
- Fixed compute allocation of 30 CPUs at 2 GB RAM per run
- DIAMOND version 0.8.3 binary as distributed with SAMSA2

## Outputs

- Table of input size versus DIAMOND annotation wall-clock runtime for each tested size point
- Runtime growth rates between successive size points
- annotation_time_scaling.csv

## How to apply

Retrieve the DIAMOND-indexed local NCBI nr Bacteria protein database and the SAMSA2-packaged sample metatranscriptome read files spanning increasing sizes. Run SAMSA2's DIAMOND annotation (DIAMOND version 0.8.3, as distributed with SAMSA2) on each sequence file in turn, holding resources constant at 30 CPUs and 2 GB RAM per run, with the database kept local and DIAMOND-indexed as SAMSA2 requires. Record wall-clock annotation time and input size (reads or bases) for every run, assemble an input-size-versus-runtime table, and compute runtime growth rates between successive size points. Verify that successive runtime increments remain approximately proportional to the corresponding input-size increments; the paper reports approximately linear scaling for DIAMOND, unlike BLAST-based pipelines which would take months at ~10 annotations per second. Save the timing table as annotation_time_scaling.csv.

## Related tools

- **DIAMOND** (Accelerated BLAST-like aligner whose annotation runtime is benchmarked against input size; run in fast mode against a local DIAMOND-indexed database) — https://github.com/bbuchfink/diamond
- **SAMSA2** (Pipeline providing the DIAMOND annotation step, packaged sample metatranscriptome files, and database download scripts used in the benchmark) — https://github.com/transcript/samsa2

## Examples

```
bash example_DIAMOND_annotation_script.bash  # run per input file at 30 CPUs / 2 GB RAM, timing each run against the DIAMOND-indexed nr Bacteria database
```

## Evaluation signals

- annotation_time_scaling.csv exists and contains a runtime entry for every input size tested
- Successive runtime increments are approximately proportional to the corresponding input-size increments (approximately linear scaling)
- All runs used identical compute resources (30 CPUs, 2 GB RAM) and the same local DIAMOND-indexed reference database
- Recorded wall-clock times are monotonic non-decreasing with input size
- Input sizes and DIAMOND version (0.8.3) are recorded for reproducibility

## Limitations

- Scaling was verified only across the tested range of input sizes; extrapolation beyond it is not validated by the paper
- DIAMOND is set by default to run in --fast mode prioritizing speed; switching to --sensitive mode may significantly increase memory requirements and change timing behavior
- Linear scaling holds only with constant compute resources and a local, DIAMOND-indexed database; BLAST-based annotation of the same data would be unacceptably slow (months at ~10 annotations per second)

## Evidence

- [methods] DIAMOND annotation of reads against a local database: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [intro] BLAST too slow for millions of sequences: "rely on BLAST [9] for annotations, which may be unacceptably slow when processing multiple millions of sequences per sample"
- [readme] DIAMOND version pinned in README: "DIAMOND, version 0.8.3: https://github.com/bbuchfink/diamond"
- [readme] DIAMOND fast mode default: [evidence span withheld — anchor status: external_doc]
- [readme] Database download step: [evidence span withheld — anchor status: external_doc]
