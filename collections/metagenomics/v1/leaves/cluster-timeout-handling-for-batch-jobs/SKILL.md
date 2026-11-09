---
name: cluster-timeout-handling-for-batch-jobs
description: Use when comparing computationally expensive R tools for longitudinal
  high-dimensional data on an HPC cluster and you need per-replicate runtime/memory
  statistics under a fixed timeout (e.g. 96 h).
when_to_use_negative:
- Single small-scale analyses where computational cost is not a bottleneck and no
  cross-tool benchmarking is intended
- Cross-sectional (non-longitudinal) data, where LongDat's longitudinal covariate-control
  pipeline does not apply
- Data types outside LongDat's scope if you need other benchmark dimensions than runtime
  and memory (the skill does not evaluate accuracy)
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: Reference method whose runtime and memory are compared against competitors;
    run via longdat_cont() on the continuous time variable
  repo: https://github.com/CCY-dev/LongDat
- name: ANCOM
  role: Competitor tool benchmarked for runtime and memory; exhibits ~350 GB memory
    consumption in the multi-covariate setting
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: lgpr
  role: Competitor tool whose runtime (~65 h with 150 samples and 1 covariate) is
    a severe bottleneck under the 96 h cluster timeout
- name: ZIBR
  role: Competitor tool benchmarked for runtime and memory; takes ~55 h in the reported
    setting
- name: R
  role: Environment in which all four tools are run and profiled
  repo: https://github.com/CCY-dev/LongDat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:b51b03f5f679594c3b39833f56d70fea52ffb9cc55ec31ce5f09b5f69e66dc11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - R
  - ZIBR
  - lgpr
  - longdat_cont()
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cluster-timeout-handling-for-batch-jobs

## Summary

A benchmarking protocol for wallclock runtime and peak-memory profiling of competing longitudinal differential-abundance tools (LongDat, ANCOM, lgpr, ZIBR) on microbiomeDASim-simulated multi-covariate data, with explicit flagging of runs exceeding a 96 h cluster timeout.

## When to use

Use when comparing computationally expensive R tools for longitudinal high-dimensional data on an HPC cluster and you need per-replicate runtime/memory statistics under a fixed timeout (e.g. 96 h).

## When NOT to use

- Single small-scale analyses where computational cost is not a bottleneck and no cross-tool benchmarking is intended
- Cross-sectional (non-longitudinal) data, where LongDat's longitudinal covariate-control pipeline does not apply
- Data types outside LongDat's scope if you need other benchmark dimensions than runtime and memory (the skill does not evaluate accuracy)

## Inputs

- Simulated longitudinal microbiome count datasets from microbiomeDASim (100 features, 50 simulations per setting, 300 samples, 16 covariates)
- LongDat_cont_master_table input (master table combining metadata and feature tables)
- Identical input datasets passed to ANCOM, lgpr and ZIBR
- Cluster timeout parameter (96 h) and per-replicate wallclock/memory profiling output

## Outputs

- Per-replicate wallclock runtime and peak memory consumption table for each tool
- Flagged list of runs exceeding the 96 h cluster timeout
- Aggregated summary table of runtime and memory statistics across 50 simulations per setting
- Comparison of measured metrics against Supplementary Table S1 and Supplementary Figs S10–S11

## How to apply

Generate simulated longitudinal microbiome count data with microbiomeDASim (e.g. 100 features, 50 simulations per setting, 300 samples, 16 covariates) and run LongDat's longdat_cont() pipeline with the continuous time variable as the reference method. Run ANCOM, lgpr and ZIBR on the identical simulated datasets under identical covariate settings. For every simulation replicate, profile each tool's wallclock runtime and peak memory consumption, and flag any run that exceeds the 96 h cluster timeout. Aggregate runtime and memory statistics across the 50 simulations per setting into a summary table, then compare the measured ANCOM memory (~350 GB), ZIBR runtime (~55 h) and lgpr runtime (~65 h with 150 samples and 1 covariate) against the values in Supplementary Table S1 and Supplementary Figs S10–S11. The rationale is that severe bottlenecks (prohibitively long lgpr runtime, ANCOM's massive memory demand) are only exposed by full per-replicate profiling under a fixed timeout, mirroring how the paper established that LongDat outperformed the alternatives in runtime and memory cost.

## Related tools

- **LongDat** (Reference method whose runtime and memory are compared against competitors; run via longdat_cont() on the continuous time variable) — https://github.com/CCY-dev/LongDat
- **ANCOM** (Competitor tool benchmarked for runtime and memory; exhibits ~350 GB memory consumption in the multi-covariate setting) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **lgpr** (Competitor tool whose runtime (~65 h with 150 samples and 1 covariate) is a severe bottleneck under the 96 h cluster timeout)
- **ZIBR** (Competitor tool benchmarked for runtime and memory; takes ~55 h in the reported setting)
- **R** (Environment in which all four tools are run and profiled) — https://github.com/CCY-dev/LongDat

## Examples

```
Rscript -e 'set.seed(123); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")'
```

## Evaluation signals

- Measured per-tool runtime and memory metrics fall within an order of magnitude of the reported bottlenecks (ANCOM ~350 GB memory, ZIBR ~55 h, lgpr ~65 h) with 96 h timeouts
- LongDat completes with lower runtime and memory than ANCOM, lgpr and ZIBR across all 50 simulations per setting
- Every replicate's metrics are recorded; any run exceeding the 96 h cluster timeout is explicitly flagged rather than silently dropped
- Summary table aggregates over all 50 simulations per setting and matches the structure of Supplementary Table S1 and Supplementary Figs S10–S11
- All tools are executed on identical simulated datasets under identical covariate settings, confirming a fair benchmark

## Limitations

- lgpr's runtime is a severe bottleneck on the multi-covariate benchmark (around 65 h with 150 samples and 1 covariate), so lgpr runs may hit the 96 h cluster timeout and require extrapolation or setting reduction
- ANCOM's ~350 GB memory demand may exceed typical node limits, so ANCOM results may be missing for large settings rather than measurable
- Reproduction is only validated within an order of magnitude of the reported bottlenecks; exact hardware-dependent runtimes will vary

## Evidence

- [abstract] LongDat outperformed the alternatives in runtime and memory cost: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [supplementary] Per-replicate profiling with 96 h timeout flagging: [evidence span withheld — anchor status: not_found]
- [supplementary] lgpr runtime bottleneck quantification: [evidence span withheld — anchor status: not_found]
- [readme] LongDat installation and data-type scope: "LongDat is able to take in several data types as input, including count, proportion, binary, ordinal and continuous data."
