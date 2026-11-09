---
name: count-data-normalization-and-rarefaction
description: Use when running longitudinal differential-abundance benchmarks on genus-level
  stool count (amplicon/metagenomic abundance) tables where the comparison across
  tools must be controlled by applying each normalization/rarefaction method consistently
  to every tool's input.
when_to_use_negative:
- Input data are proportion, binary, ordinal or continuous rather than raw counts
  — LongDat provides data_type modes for those and the count-specific normalization/rarefaction
  step does not apply
- The goal is a single-tool analysis where within-tool normalization suffices and
  cross-tool comparability on identical inputs is not required
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LongDat
  role: Run longdat_cont() on each normalized count table with data_type='count' and
    the continuous time variable as test_var; count mode requires a set seed for the
    randomized control test
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison tool run on the identical normalized count datasets to measure
    false positives under null (shuffled) time
- name: lgpr
  role: Comparison tool run on the identical normalized count datasets to measure
    false positives under null (shuffled) time
- name: ZIBR
  role: Comparison tool run on the identical normalized count datasets to measure
    false positives under null (shuffled) time
- name: R
  role: Environment in which the normalization/rarefaction variants and all tool runs
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:56501c70cdca3d16867af4a0cfc20a1ef4e39e216cfb5c16ad1bac5936aead73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  - ZIBR
  - lgpr
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# count-data-normalization-and-rarefaction

## Summary

Applies the different normalization/rarefaction strategies evaluated for count-type longitudinal microbiome data in the LongDat paper, so that downstream tools (LongDat, MaAsLin2, lgpr, ZIBR) are compared on identically processed count inputs rather than on tool-internal transformations.

## When to use

Use when running longitudinal differential-abundance benchmarks on genus-level stool count (amplicon/metagenomic abundance) tables where the comparison across tools must be controlled by applying each normalization/rarefaction method consistently to every tool's input.

## When NOT to use

- Input data are proportion, binary, ordinal or continuous rather than raw counts — LongDat provides data_type modes for those and the count-specific normalization/rarefaction step does not apply
- The goal is a single-tool analysis where within-tool normalization suffices and cross-tool comparability on identical inputs is not required

## Inputs

- Genus-level longitudinal stool microbiome count master table (individual IDs, continuous time variable, feature counts)
- Permutation seed for reproducible randomized control tests on count data
- List of normalization/rarefaction methods to evaluate for count data

## Outputs

- Normalized/rarefied versions of the count master table, one per method
- Per-tool result tables (p values, effect sizes, signals) for each normalization method
- False-positive counts per tool per normalization method on shuffled (null) time data

## How to apply

Start from a genus-level longitudinal count master table (individual, time point, feature counts). Because LongDat's data_type='count' mode includes a randomized control test that is stochastic, set a seed before running longdat_cont() so results are reproducible. Process the same count table under each normalization/rarefaction method being evaluated, then feed each identically normalized version of the data to every comparison tool — longdat_cont() with data_type='count' and the continuous time variable as test_var, plus MaAsLin2, lgpr and ZIBR — as done in the shuffled fasting-data false-positive evaluation (Supplementary Fig. S16). Tally significant calls per tool per normalization method; on shuffled (null) time data, correct processing should yield near-zero false positives for LongDat and measurably higher false-positive rates for MaAsLin2, lgpr and ZIBR.

## Related tools

- **LongDat** (Run longdat_cont() on each normalized count table with data_type='count' and the continuous time variable as test_var; count mode requires a set seed for the randomized control test) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison tool run on the identical normalized count datasets to measure false positives under null (shuffled) time)
- **lgpr** (Comparison tool run on the identical normalized count datasets to measure false positives under null (shuffled) time)
- **ZIBR** (Comparison tool run on the identical normalized count datasets to measure false positives under null (shuffled) time)
- **R** (Environment in which the normalization/rarefaction variants and all tool runs are executed)

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = fasting_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- On time-shuffled (null) count data, LongDat's false-positive count is near zero across all normalization methods, matching Supplementary Fig. S16
- On the same shuffled data, MaAsLin2, lgpr and ZIBR show measurably higher false-positive rates than LongDat
- Identical normalized inputs are confirmed to be passed to all four tools for each normalization method
- Re-running longdat_cont() with the same seed on count data reproduces the randomized control table, confirming the seed was set
- Result tables follow the LongDat schema with Signal, p-value and EffectSize columns per feature

## Limitations

- Count data in LongDat trigger a stochastic randomized control test, so results are not reproducible unless a seed is set
- This workflow targets null/false-positive calibration (shuffled time within individuals); it does not by itself quantify sensitivity on data with true associations
- Conclusions about false-positive rates are qualitative for specific normalization/rarefaction methods and specific tools (MaAsLin2, lgpr, ZIBR); ANCOM was compared in the paper but not in this shuffled fasting evaluation

## Evidence

- [intro] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [intro] test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day",: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","
- [full_text] On the shuffled semi-synthetic fasting data, LongDat showed few to no false positives in all modes, whereas MaAsLin2, ZIBR and lgpr produced higher false positive rates.: [evidence span withheld — anchor status: not_found]
- [methods] Generate null datasets by permuting (shuffling) the time variable within each individual so that all true time-feature associations are erased while individual-level structure is preserved.: [evidence span withheld — anchor status: not_found]
