---
name: compositional-count-data-tool-comparison
description: Use when selecting or validating a longitudinal differential-abundance
  method for high-dimensional microbiome count data with multiple covariates, and
  you need empirical runtime/memory/accuracy comparisons against ANCOM, lgpr, ZIBR
  or MaAsLin2.
when_to_use_negative:
- The dataset is cross-sectional with no time or repeated-measurement dimension —
  this skill is specific to longitudinal designs where time proxies treatment
- The goal is to compare tool statistical accuracy alone without profiling runtime/memory
  cost — the benchmarking machinery here centers on the 96 h timeout and memory bottlenecks
- Data are not compositional count/abundance microbiome measurements (e.g. raw RNA-seq
  or imaging data) since the compared tools assume OTU/SV-style absolute abundance
  tables
tools:
- name: LongDat
  role: 'Reference method: longdat_cont() pipeline run on the continuous time variable;
    the tool whose runtime/memory advantages are being validated'
  repo: https://github.com/CCY-dev/LongDat
- name: ANCOM
  role: Competitor tool run on identical simulated data; archive-only standalone functions
    from ancom.R with feature_table_pre_process() preprocessing
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: lgpr
  role: Competitor tool; severe runtime bottleneck (~65 h with 150 samples and 1 covariate,
    subject to 96 h timeouts)
- name: ZIBR
  role: Competitor tool; runtime bottleneck (~55 h) measured per simulation replicate
- name: MaAsLin2
  role: Competitor tool included in the accuracy/runtime/memory comparison across
    simulated and real data
- name: R
  role: Execution environment for LongDat, ANCOM, MaAsLin2 and the profiling scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:62251b6a8367d55730b789ddae63bd0fe0be14e180864042ab36e5cb5baac0ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - MaAsLin2
  - R
  - ZIBR
  - lgpr
  - longdat_cont()
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# compositional-count-data-tool-comparison

## Summary

Benchmark LongDat against competing longitudinal differential-abundance tools (MaAsLin2, ANCOM, lgpr, ZIBR) on microbiomeDASim-simulated multi-covariate count data, profiling accuracy alongside runtime and memory bottlenecks (e.g. ANCOM ~350 GB, ZIBR ~55 h, lgpr ~65 h).

## When to use

Use when selecting or validating a longitudinal differential-abundance method for high-dimensional microbiome count data with multiple covariates, and you need empirical runtime/memory/accuracy comparisons against ANCOM, lgpr, ZIBR or MaAsLin2.

## When NOT to use

- The dataset is cross-sectional with no time or repeated-measurement dimension — this skill is specific to longitudinal designs where time proxies treatment
- The goal is to compare tool statistical accuracy alone without profiling runtime/memory cost — the benchmarking machinery here centers on the 96 h timeout and memory bottlenecks
- Data are not compositional count/abundance microbiome measurements (e.g. raw RNA-seq or imaging data) since the compared tools assume OTU/SV-style absolute abundance tables

## Inputs

- Simulated longitudinal microbiome count datasets from microbiomeDASim (100 features, 50 simulations per setting, varied sample sizes and covariate counts)
- LongDat master table (LongDat_cont_master_table format: metadata + feature table)
- ANCOM inputs: absolute-abundance OTU/feature table and metadata processed via feature_table_pre_process()
- Supplementary Table S1 and Supplementary Figs S10-S11 as reference benchmark values

## Outputs

- Per-tool wallclock runtime and peak memory statistics aggregated across 50 simulations per setting (summary table)
- Flagged replicates exceeding the 96 h cluster timeout
- Per-tool differential-abundance result tables (e.g. LongDat result table with Signal, EffectSize, p values; ANCOM W-statistic table)
- Reproduction verdict comparing measured metrics to reported bottlenecks (ANCOM ~350 GB, ZIBR ~55 h, lgpr ~65 h)

## How to apply

Generate simulated longitudinal microbiome count datasets with microbiomeDASim under a factorial design (e.g. 100 features, 50 simulations per setting, up to 300 samples and 16 covariates), and run LongDat's longdat_cont() pipeline on the continuous time variable (the proxy for treatment) as the reference method in R. Run each competitor (ANCOM from the archived ancom.R workflow, lgpr, ZIBR, MaAsLin2) on the identical simulated datasets under matched covariate settings. Profile each tool's wallclock runtime and peak memory per replicate, flagging runs exceeding a 96 h cluster timeout, and aggregate across the 50 simulations per setting into a summary table. Compare measured values (ANCOM ~350 GB memory, ZIBR ~55 h, lgpr ~65 h with 150 samples and 1 covariate) against Supplementary Table S1 and Supplementary Figs S10-S11, and accept the reproduction when metrics fall within an order of magnitude of the reported bottlenecks and LongDat completes with lower runtime and memory than all competitors.

## Related tools

- **LongDat** (Reference method: longdat_cont() pipeline run on the continuous time variable; the tool whose runtime/memory advantages are being validated) — https://github.com/CCY-dev/LongDat
- **ANCOM** (Competitor tool run on identical simulated data; archive-only standalone functions from ancom.R with feature_table_pre_process() preprocessing) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **lgpr** (Competitor tool; severe runtime bottleneck (~65 h with 150 samples and 1 covariate, subject to 96 h timeouts))
- **ZIBR** (Competitor tool; runtime bottleneck (~55 h) measured per simulation replicate)
- **MaAsLin2** (Competitor tool included in the accuracy/runtime/memory comparison across simulated and real data)
- **R** (Execution environment for LongDat, ANCOM, MaAsLin2 and the profiling scripts)

## Examples

```
install.packages("LongDat"); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Measured per-tool runtime and memory fall within an order of magnitude of the reported bottlenecks: ANCOM ~350 GB memory, ZIBR ~55 h, lgpr ~65 h (with 150 samples and 1 covariate)
- LongDat completes with lower runtime and memory than MaAsLin2, ANCOM, lgpr and ZIBR on every multi-covariate setting
- Aggregated runtime/memory statistics across the 50 simulations per setting reproduce Supplementary Table S1 and Supplementary Figs S10-S11 trends
- Runs exceeding the 96 h cluster timeout are flagged and counted per tool, with lgpr and ZIBR dominating the timeout list
- LongDat results include the expected schema: result table with Signal (e.g. OK_nc) and EffectSize columns, covariate table, and randomized control table for count data

## Limitations

- lgpr's runtime is a severe bottleneck on the multi-covariate benchmark and can exceed a 96 h cluster timeout, so competitor runs may not finish
- The standalone ANCOM functions in FrederickHuangLin/ANCOM-Code-Archive are no longer maintained; users are directed to the ANCOMBC package (ancom/ancombc functions)
- Benchmark conclusions are grounded in microbiomeDASim-simulated data and the paper's settings (e.g. ~350 GB ANCOM memory at large sample sizes); they may not transfer to other data scales or types
- LongDat install may fail with namespace export errors (e.g. object not exported by 'namespace:B_package'); install the failing dependency first

## Evidence

- [abstract] We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data.: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
- [abstract] LongDat outperformed competitors: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [readme] ANCOM archive is unmaintained: [evidence span withheld — anchor status: external_doc]
- [readme] LongDat install from CRAN: [evidence span withheld — anchor status: not_found]
