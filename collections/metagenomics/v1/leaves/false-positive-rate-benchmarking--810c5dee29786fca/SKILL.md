---
name: false-positive-rate-benchmarking
description: Use when validating a longitudinal association pipeline on microbiome
  count data by measuring false-positive rates against a permuted-time null in which
  all true time-feature associations are eliminated.
when_to_use_negative:
- When the goal is detecting true time-feature associations on unshuffled data rather
  than estimating false-positive control on a null
- When the dataset is cross-sectional with no repeated measures per individual, since
  within-individual time shuffling is undefined
- When data are not count-type microbiome features, since the comparison evaluates
  count-data normalization/rarefaction methods
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: Tested tool run via longdat_cont() on shuffled-time null data with data_type
    = 'count' to measure its false positive rate
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison tool run on identical shuffled datasets to benchmark its false
    positive rate against LongDat
- name: lgpr
  role: Comparison tool run on identical shuffled datasets to benchmark its false
    positive rate against LongDat
- name: ZIBR
  role: Comparison tool run on identical shuffled datasets to benchmark its false
    positive rate against LongDat
- name: R
  role: Environment for shuffling the time variable within each individual and running
    longdat_cont()
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:26aa14a9724f41901cbe17bc39b34c69ad63dfb296170c530ca8c3e87a3ab5cf
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# false-positive-rate-benchmarking

## Summary

Benchmark false-positive control of longitudinal high-dimensional data tools (LongDat, MaAsLin2, lgpr, ZIBR) by shuffling the time variable within each individual to erase all true associations, then tallying significant calls per tool. This validates that a pipeline's reported associations are not artifacts of individual-level structure.

## When to use

Use when validating a longitudinal association pipeline on microbiome count data by measuring false-positive rates against a permuted-time null in which all true time-feature associations are eliminated.

## When NOT to use

- When the goal is detecting true time-feature associations on unshuffled data rather than estimating false-positive control on a null
- When the dataset is cross-sectional with no repeated measures per individual, since within-individual time shuffling is undefined
- When data are not count-type microbiome features, since the comparison evaluates count-data normalization/rarefaction methods

## Inputs

- Genus-level fasting stool microbiome longitudinal master table in R (individual_id, continuous time variable, feature counts)
- Shuffled/permuted-time null datasets with time permuted within each individual
- Seed value for reproducible randomized control test with count data

## Outputs

- Per-tool, per-normalization-method false positive tallies (features significant under permuted time)
- Result tables from longdat_cont() with Signal column, p values and effect sizes
- Comparison of false positive rates matching the qualitative pattern of Supplementary Fig. S16

## How to apply

Load the genus-level fasting stool microbiome longitudinal master table in R and generate null datasets by permuting the time variable within each individual, preserving individual-level structure while erasing all true time-feature associations. Run longdat_cont() on each shuffled dataset with data_type = 'count' and the continuous time variable as test_var, setting a seed beforehand so the randomized control test is reproducible. Run comparison tools (MaAsLin2, lgpr, ZIBR) on the identical shuffled datasets, repeating across the different normalization/rarefaction methods evaluated for count data. Tally false positives (features called significant despite permuted time) per tool per normalization method and compare against Supplementary Fig. S16 expectations.

## Related tools

- **LongDat** (Tested tool run via longdat_cont() on shuffled-time null data with data_type = 'count' to measure its false positive rate) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison tool run on identical shuffled datasets to benchmark its false positive rate against LongDat)
- **lgpr** (Comparison tool run on identical shuffled datasets to benchmark its false positive rate against LongDat)
- **ZIBR** (Comparison tool run on identical shuffled datasets to benchmark its false positive rate against LongDat)
- **R** (Environment for shuffling the time variable within each individual and running longdat_cont())

## Examples

```
set.seed(2023); test_cont <- longdat_cont(input = shuffled_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Near-zero false positive rate for LongDat on shuffled-time data (few to no features called significant)
- Measurably higher false positive rates for MaAsLin2, lgpr and ZIBR than LongDat on the identical shuffled datasets
- Qualitative pattern matches Supplementary Fig. S16
- Set seed before running longdat_cont() with data_type = 'count' so the randomized control test is reproducible
- Individual-level structure is preserved in the null datasets (time shuffled only within each individual)

## Limitations

- Result is a qualitative comparison on semi-synthetic fasting stool microbiome data; the false-positive ordering of tools may not transfer to other datasets or data types
- With count data_type, a seed must be set or the randomized control test is not reproducible
- Shuffling only erases time-feature associations; covariate structure and individual effects remain, so false positives may reflect covariate confounding rather than time modeling alone

## Evidence

- [full_text] On the shuffled semi-synthetic fasting data, LongDat showed few to no false positives in all modes, whereas MaAsLin2, ZIBR and lgpr produced higher false positive rates.: [evidence span withheld — anchor status: not_found]
- [methods] Generate null datasets by permuting (shuffling) the time variable within each individual so that all true time-feature associations are erased while individual-level structure is preserved.: [evidence span withheld — anchor status: not_found]
- [methods] Run longdat_cont() on each shuffled dataset with data_type = 'count' and the continuous time variable as test_var, setting a seed beforehand so the randomized control test is reproducible.: [evidence span withheld — anchor status: not_found]
- [intro] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [abstract] We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data.: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
