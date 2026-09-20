---
name: computational-cost-metrics-aggregation
description: Use when per-assembler computational cost (runtime, CPU/hour, max memory,
  data read/written) must be extracted from deposited LMAS run outputs and compared
  for consistency across repeated runs on the same sample.
when_to_use_negative:
- The goal is assembly-quality evaluation (N50, misassemblies, reference alignment)
  rather than computational cost — use the LMAS report's assembly evaluation metrics
  instead
- Runs were not executed via LMAS, so the standardized per-assembler execution metrics
  (version, container, CPU/hour, memory) were not captured
- Only a single run per assembler is available, making the cross-run consistency comparison
  that motivated this aggregation meaningless
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0092
tools:
- name: LMAS
  role: Source of the per-assembler computational resource metrics (Performance tab)
    and containerized workflow whose deposited outputs are parsed
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: Assembler whose inconsistent runtime and memory allocation across repeated
    runs is detected and compiled by this aggregation
- name: MetaHipmer2
  role: Assembler whose inconsistent resource requirements across repeated LMAS runs
    are detected and compiled by this aggregation
- name: minia
  role: Benchmarked assembler included as a row in the aggregated resource_usage_table
  repo: https://github.com/B-UMMI/LMAS
- name: VelvetOptimiser
  role: Benchmarked assembler included as a row in the aggregated resource_usage_table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:f94663187c1e23c2b95aac62dcb0453284b7318ea5876eb2656674716c2c4cb7
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
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# computational-cost-metrics-aggregation

## Summary

Compiles the five computational resource metrics (average execution time, CPU/hour, maximum memory in GB, average data read, and average data written) that LMAS registers per assembler, aggregated across repeated runs on the ZymoBIOMICS sample, into a single per-assembler table for cost/reproducibility assessment.

## When to use

Use when per-assembler computational cost (runtime, CPU/hour, max memory, data read/written) must be extracted from deposited LMAS run outputs and compared for consistency across repeated runs on the same sample.

## When NOT to use

- The goal is assembly-quality evaluation (N50, misassemblies, reference alignment) rather than computational cost — use the LMAS report's assembly evaluation metrics instead
- Runs were not executed via LMAS, so the standardized per-assembler execution metrics (version, container, CPU/hour, memory) were not captured
- Only a single run per assembler is available, making the cross-run consistency comparison that motivated this aggregation meaningless

## Inputs

- Deposited LMAS run output data for the ZymoBIOMICS dataset (Zenodo/GigaDB deposit, e.g. Zenodo record 7129554)
- Per-assembler LMAS performance records (assembler version, container, execution metrics)
- LMAS interactive HTML report Performance tab definitions (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB))

## Outputs

- resource_usage_table.csv with one row per assembler and one column per metric (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB))
- Per-assembler consistency assessment across the 3 repeated LMAS runs (e.g. inconsistent runtimes/memory for ABySS and MetaHipmer2)

## How to apply

Retrieve the deposited LMAS run output data for the ZymoBIOMICS dataset from the associated Zenodo/GigaDB deposit, then parse the per-assembler performance records (version, container, execution metrics) captured by LMAS for each benchmarked assembler. Extract the five metrics defined in the LMAS report Performance tab: average execution time (hours), CPU/hour, maximum memory (GB), average data read (GB), and average data written (GB). Aggregate each metric at the assembler level across the deposited runs — the study ran the LMAS workflow 3 times for all samples with default parameters — and write the result to resource_usage_table.csv with one row per assembler and one column per metric. Use the aggregated table as a decision point: assemblers showing inconsistent resource requirements across repeated runs (as observed for ABySS and MetaHipmer2) can be flagged and excluded from subsequent analyses.

## Related tools

- **LMAS** (Source of the per-assembler computational resource metrics (Performance tab) and containerized workflow whose deposited outputs are parsed) — https://github.com/B-UMMI/LMAS
- **ABySS** (Assembler whose inconsistent runtime and memory allocation across repeated runs is detected and compiled by this aggregation)
- **MetaHipmer2** (Assembler whose inconsistent resource requirements across repeated LMAS runs are detected and compiled by this aggregation)
- **minia** (Benchmarked assembler included as a row in the aggregated resource_usage_table) — https://github.com/B-UMMI/LMAS
- **VelvetOptimiser** (Benchmarked assembler included as a row in the aggregated resource_usage_table)

## Evaluation signals

- resource_usage_table.csv contains a row for every assembler in the LMAS benchmark
- All five performance metric fields are present as columns: Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB), as defined in the LMAS report Performance tab
- Metric values are aggregated over the 3 repeated LMAS runs per sample, matching the study design (workflow run 3 times with default parameters)
- Aggregated values reproduce the observed inconsistency findings: ABySS and MetaHipmer2 differ in runtime and memory allocation across runs, supporting their exclusion from subsequent analyses
- Each metric is expressed in the units defined by LMAS (hours for time, GB for memory and data read/written)

## Limitations

- LMAS is computationally intensive due to the de novo assembly process, so repeated runs required for consistency checking carry a high computational cost
- ABySS and MetaHipmer2 showed inconsistent resource requirements across repeated runs on the same sample, limiting the reliability of their cost figures
- Resource requirements were measured with default parameters only; usability under non-default parameter tuning was not explored
- ABySS failed to produce an assembly for sample ERR2984773 in 1 of the runs, so per-run metric completeness cannot be assumed for all assemblers

## Evidence

- [results] ABySS and MetaHipmer2 showed inconsistent resource requirements across repeated LMAS runs on the same sample, differing in runtime and memory allocation, which contributed to their exclusion from subsequent analyses.: "the following assemblers have not been included in subsequent analyses: ABySS, MetaHipmer2, minia, and VelvetOptimiser"
- [abstract] k-mer De Bruijn graph assemblers outperformed the alternative approaches but came with a greater computational cost.: "Overall, in our test datasets, k-mer De Bruijn graph assemblers outperformed the alternative approaches but came with a greater computational cost."
- [discussion] LMAS is computationally intensive due to the de novo assembly process.: "Although computationally intensive due to the complex nature of the de novo assembly process, LMAS is the only software-integrating assembly and its evaluation into a single pipeline"
- [results] The LMAS workflow was run 3 times for all samples using default parameters.: "the LMAS workflow was run 3 times for all samples using default parameters"
- [other] Validation: confirm the output table contains a row for every assembler in the LMAS benchmark and includes all five performance metric fields (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)) as defined in the LMAS report Performance tab.: [evidence span withheld — anchor status: not_found]
