---
name: continuous-time-covariate-modeling
description: Use when you have a longitudinal master table (metadata + feature table)
  with a continuous time variable such as 'Day' and count or continuous feature data,
  and need to detect features changing significantly over time while disentangling
  covariate effects.
when_to_use_negative:
- Time variable is discrete (e.g. before/after treatment time points) — use longdat_disc()
  with a LongDat_disc_master_table instead
- Data are cross-sectional with no longitudinal time dimension — ANCOM-style cross-sectional
  analysis is the appropriate design
- Input is raw unprocessed abundance; for ANCOM comparisons the feature table must
  be absolute abundance preprocessed with feature_table_pre_process (zero_cut = 0.90,
  lib_cut), not relative abundance
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3697
tools:
- name: LongDat
  role: Primary R package providing longdat_cont(), make_master_table() and cuneiform_plot()
    for covariate-sensitive continuous-time longitudinal modeling
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Runtime environment in which longdat_cont() is executed (install.packages('LongDat')
    from CRAN)
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison tool that drops to only 2 significant species with medication covariates
    in the fasting MetS cohort vs LongDat's robust result
  repo: https://github.com/biobakery/biobakery
- name: ANCOM
  role: Competitor tool exhibiting ~350 GB memory consumption on the multi-covariate
    simulation benchmark
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: lgpr
  role: Competitor tool with ~65 h runtime (150 samples, 1 covariate) on the microbiomeDASim
    benchmark
- name: ZIBR
  role: Competitor tool with ~55 h runtime on the multi-covariate simulation benchmark
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:6a4161cdb199a9c7e9e35bfc1ac087b494df1ef6f7985a288aafd04ba39dbde8
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
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# continuous-time-covariate-modeling

## Summary

Model high-dimensional longitudinal count/continuous data against a continuous time variable (proxy for treatment) with longdat_cont(), which loops over one covariate per model so multiple covariates are handled robustly, and interpret Signal/EffectSize columns plus the covariate table to separate time effects from covariate effects.

## When to use

Use when you have a longitudinal master table (metadata + feature table) with a continuous time variable such as 'Day' and count or continuous feature data, and need to detect features changing significantly over time while disentangling covariate effects.

## When NOT to use

- Time variable is discrete (e.g. before/after treatment time points) — use longdat_disc() with a LongDat_disc_master_table instead
- Data are cross-sectional with no longitudinal time dimension — ANCOM-style cross-sectional analysis is the appropriate design
- Input is raw unprocessed abundance; for ANCOM comparisons the feature table must be absolute abundance preprocessed with feature_table_pre_process (zero_cut = 0.90, lib_cut), not relative abundance

## Inputs

- LongDat_cont_master_table (data frame combining metadata and feature columns, e.g. 2 time points day 0 and 7)
- LongDat_cont_metadata_table (sample metadata incl. time and covariates)
- LongDat_cont_feature_table (feature abundance matrix)
- data_type = 'count' | 'proportion' | 'binary' | 'ordinal' | 'continuous'
- Continuous test variable name (e.g. 'Day') and optional covariate columns (e.g. medication)

## Outputs

- Result table with p values, Signal (e.g. OK_nc, OK_nrc, NS), and EffectSize per feature
- Covariate table (blank when no covariate effect is detected)
- Randomized control table (for data_type = 'count') or Normalize_method table
- Optional cuneiform_plot() visualization of the result table

## How to apply

Assemble a master table from the metadata and feature tables with make_master_table(), then run longdat_cont(input = master_table, data_type = 'count', test_var = 'Day') on the continuous time variable, which serves as the proxy for treatment. Set a seed before running when data_type equals 'count' so the randomized control test is reproducible. Unlike MaAsLin2, which errors on models with multiple fixed effects, LongDat loops over one covariate per model, so include medication or other covariates directly and inspect the Signal column ('OK_nc' = significant time change with no covariate; 'OK_nrc' = time effect not reducible to covariate; 'NS' = no time effect) to see which time-associated features are reducible to covariates. Read EffectSize for direction and magnitude (e.g. BacteriumA -0.78 decreasing, BacteriumC 0.71 increasing) and check the covariate table, which is blank when no covariate effect is found. Benchmarking context: on multi-covariate microbiomeDASim simulations (100 features, 50 simulations, 300 samples, 16 covariates), competitors show prohibitive costs — ANCOM ~350 GB memory, ZIBR ~55 h and lgpr ~65 h runtime vs a 96 h cluster timeout — so LongDat is the reference choice when covariate count is large.

## Related tools

- **LongDat** (Primary R package providing longdat_cont(), make_master_table() and cuneiform_plot() for covariate-sensitive continuous-time longitudinal modeling) — https://github.com/CCY-dev/LongDat
- **R** (Runtime environment in which longdat_cont() is executed (install.packages('LongDat') from CRAN)) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison tool that drops to only 2 significant species with medication covariates in the fasting MetS cohort vs LongDat's robust result) — https://github.com/biobakery/biobakery
- **ANCOM** (Competitor tool exhibiting ~350 GB memory consumption on the multi-covariate simulation benchmark) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **lgpr** (Competitor tool with ~65 h runtime (150 samples, 1 covariate) on the microbiomeDASim benchmark)
- **ZIBR** (Competitor tool with ~55 h runtime on the multi-covariate simulation benchmark)

## Examples

```
test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day", test_time_order = FALSE)
```

## Evaluation signals

- Signal column values are interpretable: significant time predictors show 'OK_nc' (or 'OK_nrc' in longdat_disc), non-significant features show 'NS'
- EffectSize signs match the direction of change (negative = decreasing over time, positive = increasing; e.g. -0.78 and 0.71 in the dummy example)
- Covariate table is blank exactly when no covariate effect is found in the result table
- With data_type = 'count', the randomized control test is reproducible across runs because a seed was set before calling longdat_cont()
- On the Maifeld fasting MetS reanalysis, expect ~27 significant species per tool without covariates (19 overlapping) and only 2 for MaAsLin2 with covariates, while LongDat retains covariate-disentangled results

## Limitations

- When data_type equals 'count', a seed must be set or the randomized control test is not reproducible
- Package installation can fail with namespace export errors (e.g. object 'A' is not exported by 'namespace:B_package'); install the dependency first
- Competitor comparison numbers are order-of-magnitude benchmarks: ANCOM ~350 GB memory, ZIBR ~55 h, lgpr ~65 h can exceed a 96 h cluster timeout, so scale carefully on shared clusters

## Evidence

- [intro] analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at the same time: "analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at the same time"
- [intro] set seed for reproducible randomized control test: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] BacteriumA decreasing, BacteriumC increasing with effect sizes: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively."
- [supplementary] LongDat handles multiple covariates by looping over one covariate per model: [evidence span withheld — anchor status: not_found]
- [abstract] LongDat outperformed the alternative tools in accuracy, runtime and memory: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [results] signal OK_nrc meaning time effect independent of covariates: "treatment is a significant predictor for BacteriumA as it shows "OK_nrc" (which represents "OK and not reducible to covariate")"
- [intro] make_master_table creates master table from metadata and feature tables: "The function make_master_table() helps you to create master table from metadata and feature tables."
- [readme] Installation namespace error remedy: [evidence span withheld — anchor status: external_doc]
