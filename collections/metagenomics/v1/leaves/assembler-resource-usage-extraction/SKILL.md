---
name: assembler-resource-usage-extraction
description: Use when you have deposited LMAS benchmark output (e.g. Zenodo/GigaDB
  deposit for giac122) for repeated runs and need per-assembler runtime, CPU/hour,
  memory, and data read/written statistics, or must justify excluding assemblers on
  resource grounds.
when_to_use_negative:
- When you only need assembly-quality metrics (N50, misassembly categories, contigs
  >1 kbp) rather than computational cost — use the LMAS report's assembly-evaluation
  outputs instead.
- When runs used non-default assembler parameters — the article notes assembler usability
  was assessed only with the default parameters used, so resource figures are not
  transferable to tuned settings.
- When the deposit lacks per-run resource records (e.g. a single collapsed report),
  aggregation across repeated runs is impossible and consistency cannot be assessed.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Automated Nextflow workflow that registers per-assembler execution metrics
    (version, container, runtime, memory, data read/written) and exposes them in the
    report Performance tab
  repo: https://github.com/B-UMMI/LMAS
- name: ABySS
  role: Benchmarked assembler whose resource usage showed inconsistency across repeated
    LMAS runs, contributing to its exclusion from subsequent analyses
  repo: https://github.com/BirolLab/abyss
- name: MetaHipmer2
  role: Benchmarked assembler with inconsistent runtime and memory allocation across
    repeated LMAS runs, contributing to its exclusion from subsequent analyses
- name: minia
  role: Benchmarked assembler included in the per-assembler resource-usage aggregation;
    performed poorly and was excluded from subsequent analyses
  repo: https://github.com/GATB/minia
- name: VelvetOptimiser
  role: Benchmarked assembler included in the per-assembler resource-usage aggregation;
    performed poorly and was excluded from subsequent analyses
  repo: https://github.com/tseemann/VelvetOptimiser
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:a6faa18c8fbf759afe79a028b3e6a7794831a2b13fba54d5ed0e6c6dd1a64be0
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# assembler-resource-usage-extraction

## Summary

Extract and tabulate the five computational resource metrics (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)) that the LMAS Nextflow pipeline registers per assembler, to expose which assemblers have inconsistent or prohibitive resource requirements across repeated runs on a ZymoBIOMICS sample.

## When to use

Use when you have deposited LMAS benchmark output (e.g. Zenodo/GigaDB deposit for giac122) for repeated runs and need per-assembler runtime, CPU/hour, memory, and data read/written statistics, or must justify excluding assemblers on resource grounds.

## When NOT to use

- When you only need assembly-quality metrics (N50, misassembly categories, contigs >1 kbp) rather than computational cost — use the LMAS report's assembly-evaluation outputs instead.
- When runs used non-default assembler parameters — the article notes assembler usability was assessed only with the default parameters used, so resource figures are not transferable to tuned settings.
- When the deposit lacks per-run resource records (e.g. a single collapsed report), aggregation across repeated runs is impossible and consistency cannot be assessed.

## Inputs

- Deposited LMAS run output data for the ZymoBIOMICS dataset (Zenodo deposit 10.5281/zenodo.7129554 / GigaDB)
- Per-assembler LMAS performance records from the LMAS report Performance tab (version, container, execution metrics)
- Repeated-run identifiers (LMAS workflow run 3 times with default parameters per sample)

## Outputs

- resource_usage_table.csv with one row per assembler and one column per metric (Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB))

## How to apply

Retrieve the deposited LMAS run output data for the ZymoBIOMICS dataset from the Zenodo/GigaDB deposit associated with the study, then parse the per-assembler performance records (version, container, execution metrics) captured by LMAS for each benchmarked assembler. From the LMAS report Performance tab, extract the five recorded metrics: average execution time (hours), CPU/hour, maximum memory (GB), average data read (GB), and average data written (GB). Aggregate each metric at the assembler level across the three repeated LMAS runs (the workflow was run 3 times for all samples using default parameters). Compare metric consistency within each assembler across runs: in the source study, ABySS and MetaHipmer2 showed divergent runtime and memory allocation across repeated runs and were excluded from subsequent analyses, alongside ABySS failing one run. Write the compiled metrics to a single CSV (one row per assembler, one column per metric) and validate that every benchmarked assembler appears and all five metric fields are populated.

## Related tools

- **LMAS** (Automated Nextflow workflow that registers per-assembler execution metrics (version, container, runtime, memory, data read/written) and exposes them in the report Performance tab) — https://github.com/B-UMMI/LMAS
- **ABySS** (Benchmarked assembler whose resource usage showed inconsistency across repeated LMAS runs, contributing to its exclusion from subsequent analyses) — https://github.com/BirolLab/abyss
- **MetaHipmer2** (Benchmarked assembler with inconsistent runtime and memory allocation across repeated LMAS runs, contributing to its exclusion from subsequent analyses)
- **minia** (Benchmarked assembler included in the per-assembler resource-usage aggregation; performed poorly and was excluded from subsequent analyses) — https://github.com/GATB/minia
- **VelvetOptimiser** (Benchmarked assembler included in the per-assembler resource-usage aggregation; performed poorly and was excluded from subsequent analyses) — https://github.com/tseemann/VelvetOptimiser

## Evaluation signals

- resource_usage_table.csv contains exactly one row for every assembler in the LMAS benchmark (including ABySS, MetaHipmer2, minia, VelvetOptimiser)
- All five metric columns are present and named as in the LMAS report Performance tab: Avg Time, CPU/Hour, Max Memory (GB), Average Read (GB), Average Write (GB)
- Metric values are aggregated over the 3 repeated LMAS runs per assembler, not a single run
- Cross-run spread within an assembler is explicitly reported; ABySS and MetaHipmer2 are expected to show divergent runtime and memory values across runs
- Resource figures are consistent with the qualitative finding that k-mer De Bruijn graph assemblers carried a greater computational cost than alternative approaches

## Limitations

- ABySS failed to produce an assembly for sample ERR2984773 in 1 of the runs, so resource rows may be missing for that run and should not be silently imputed
- Resource statistics reflect default parameters only; non-default parameter tuning was not explored, and the article notes usability 'may be limited, at least with the default parameters we used'
- Findings derive from the ZymoBIOMICS Microbial Community Standards and other defined mock communities, which 'might not be representative of the metagenomic complexity of the samples of interest of most researchers'

## Evidence

- [results] the LMAS workflow was run 3 times for all samples using default parameters: "the LMAS workflow was run 3 times for all samples using default parameters"
- [results] ABySS run failure: "ABySS failed to produce an assembly for sample ERR2984773 for 1 of the runs"
- [results] Exclusion of poor performers: "the following assemblers have not been included in subsequent analyses: ABySS, MetaHipmer2, minia, and VelvetOptimiser"
- [abstract] Computational cost finding: "k-mer De Bruijn graph assemblers outperformed the alternative approaches but came with a greater computational cost."
- [readme] LMAS description from README: "LMAS is an automated workflow enabling the benchmarking of traditional and metagenomic prokaryotic de novo assembly software using defined mock communities."
