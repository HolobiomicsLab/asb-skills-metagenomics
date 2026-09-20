---
name: benchmark-performance-tabulation
description: Use when you need to reproduce or audit per-assembler computational cost
  from deposited LMAS benchmark outputs — e.g. comparing runtime, memory, CPU/hour,
  and data read/written across assemblers or checking consistency across repeated
  runs of the same mock community sample.
when_to_use_negative:
- The goal is assembly quality assessment (N50, misassemblies, reference-based metrics)
  rather than computational resource accounting
- Only a single LMAS run is available per sample, so run-to-run consistency of resource
  requirements cannot be evaluated
- The pipeline outputs were not produced by LMAS (the Performance tab metrics are
  LMAS-specific and not registered by other assembly benchmarks)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LMAS
  role: Automated benchmarking workflow that registers per-assembler execution metrics
    reported in the interactive report's Performance tab
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: Benchmarked assembler whose resource metrics are tabulated; showed inconsistent
    runtime and memory across repeated LMAS runs
- name: MetaHipmer2
  role: Benchmarked assembler whose resource metrics are tabulated; showed inconsistent
    runtime and memory across repeated LMAS runs
- name: minia
  role: Benchmarked assembler whose resource metrics are tabulated and included in
    the resource usage table
- name: VelvetOptimiser
  role: Benchmarked assembler whose resource metrics are tabulated and included in
    the resource usage table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:91288ca8ae1990c16434149c31816471287b9ec6d3c0eea0d482061eda424751
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - ABySS
  - LMAS
  - MetaHipmer2
  - VelvetOptimiser
  - minia
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# benchmark-performance-tabulation

## Summary

Tabulates the five computational resource metrics (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)) that the LMAS report's Performance tab registers per assembler, aggregated across repeated runs on the same sample (e.g. ZymoBIOMICS), to expose run-to-run variability in resource requirements.

## When to use

Use when you need to reproduce or audit per-assembler computational cost from deposited LMAS benchmark outputs — e.g. comparing runtime, memory, CPU/hour, and data read/written across assemblers or checking consistency across repeated runs of the same mock community sample.

## When NOT to use

- The goal is assembly quality assessment (N50, misassemblies, reference-based metrics) rather than computational resource accounting
- Only a single LMAS run is available per sample, so run-to-run consistency of resource requirements cannot be evaluated
- The pipeline outputs were not produced by LMAS (the Performance tab metrics are LMAS-specific and not registered by other assembly benchmarks)

## Inputs

- Deposited LMAS run output records for a sample (e.g. ZymoBIOMICS) from Zenodo/GigaDB, containing per-assembler version, container, and execution metrics
- List of benchmarked assemblers (e.g. ABySS, MetaHipmer2, minia, VelvetOptimiser, and the better performers)
- LMAS report Performance tab field definitions (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB))

## Outputs

- resource_usage_table.csv with one row per assembler and one column per metric: Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)

## How to apply

Retrieve the deposited LMAS run outputs (Zenodo/GigaDB) for the benchmarked sample, e.g. the ZymoBIOMICS Microbial Community Standards dataset for which the workflow was run 3 times with default parameters. Parse the per-assembler performance records LMAS captured (assembler version, container, execution metrics) for each benchmarked assembler. Extract the five metrics defined in the LMAS report Performance tab: average execution time (hours), CPU/hour, maximum memory (GB), average data read (GB), and average data written (GB). Aggregate each metric at the assembler level across the repeated runs, then write a single CSV (resource_usage_table.csv) with one row per assembler and one column per metric. Rationale: the paper found resource usage is a key differentiator — k-mer De Bruijn graph assemblers outperformed alternatives but at greater computational cost — and ABySS and MetaHipmer2 showed inconsistent resource requirements across repeats, which motivated their exclusion from subsequent analyses.

## Related tools

- **LMAS** (Automated benchmarking workflow that registers per-assembler execution metrics reported in the interactive report's Performance tab) — https://github.com/B-UMMI/LMAS
- **ABySS** (Benchmarked assembler whose resource metrics are tabulated; showed inconsistent runtime and memory across repeated LMAS runs)
- **MetaHipmer2** (Benchmarked assembler whose resource metrics are tabulated; showed inconsistent runtime and memory across repeated LMAS runs)
- **minia** (Benchmarked assembler whose resource metrics are tabulated and included in the resource usage table)
- **VelvetOptimiser** (Benchmarked assembler whose resource metrics are tabulated and included in the resource usage table)

## Examples

```
python -c "import pandas as pd; df.groupby('assembler')[['Avg Time','CPU/Hour','Max Memory (GB)','Average Read (GB)','Average Write (GB)']].mean().to_csv('resource_usage_table.csv')"
```

## Evaluation signals

- resource_usage_table.csv contains one row for every assembler in the LMAS benchmark
- The table includes all five performance metric fields exactly as defined in the LMAS report Performance tab: Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)
- Metric values are aggregated across the 3 repeated LMAS runs on the same sample rather than reported per run
- For ABySS and MetaHipmer2, runtime and memory values are expected to vary across the repeated runs, consistent with the reported inconsistency
- Any run missing an assembly (e.g. ABySS on ERR2984773 in 1 of 3 runs) is handled explicitly rather than silently imputed

## Limitations

- ABySS and MetaHipmer2 show inconsistent resource requirements across repeated runs on the same sample (differing runtime and memory allocation), so their aggregated values are not stable reference points
- ABySS failed to produce an assembly for sample ERR2984773 in 1 of the 3 runs, so some per-run records may be absent
- Resource observations reflect LMAS default parameters only; non-default parameter tuning was not explored

## Evidence

- [abstract] k-mer De Bruijn graph assemblers outperformed alternative approaches but came with a greater computational cost: "Overall, in our test datasets, k-mer De Bruijn graph assemblers outperformed the alternative approaches but came with a greater computational cost."
- [results] LMAS workflow run 3 times with default parameters: "the LMAS workflow was run 3 times for all samples using default parameters"
- [results] ABySS failed one run on ERR2984773: "ABySS failed to produce an assembly for sample ERR2984773 for 1 of the runs"
- [discussion] LMAS is computationally intensive: "Although computationally intensive due to the complex nature of the de novo assembly process, LMAS is the only software-integrating assembly and its evaluation into a single pipeline"
- [readme] LMAS report presents interactive performance metrics: "LMAS is an automated workflow enabling the benchmarking of traditional and metagenomic prokaryotic de novo assembly software using defined mock communities."
