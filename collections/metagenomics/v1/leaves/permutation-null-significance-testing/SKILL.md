---
name: permutation-null-significance-testing
description: Use when validating a longitudinal tool's false-positive control on semi-synthetic
  or real count/proportion data by shuffling the continuous time variable within each
  individual, as done for the fasting stool microbiome benchmark.
when_to_use_negative:
- The data are not longitudinal or lack a within-individual time variable that can
  be permuted to create a valid null
- The goal is to detect real time/treatment effects rather than to benchmark false-positive
  control under a null hypothesis
- The analysis is on data types where LongDat's randomized control table is not produced
  (e.g. measurement/others data return a Normalize_method table instead)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3053
tools:
- name: LongDat
  role: longdat_cont() with data_type='count' and a continuous test_var; produces
    the result table plus a randomized control table on shuffled (null) data
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: comparison tool run on the identical shuffled datasets to measure its false-positive
    rate
- name: lgpr
  role: comparison tool run on the identical shuffled datasets to measure its false-positive
    rate
- name: ZIBR
  role: comparison tool run on the identical shuffled datasets to measure its false-positive
    rate
- name: R
  role: implementation environment for the permutation scheme, seed setting and tool
    runs
  repo: https://github.com/CCY-dev/LongDat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:c11dcdef059d23b71866a11fe1db6b81758d9357e856e1e425101c64d3240f45
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# permutation-null-significance-testing

## Summary

A null-control evaluation strategy for longitudinal high-dimensional analyses in which the time variable is permuted within each individual (LongDat's randomized control test) so all true time-feature associations are erased, allowing false-positive rates of LongDat versus MaAsLin2, lgpr and ZIBR to be measured on count-type microbiome data.

## When to use

Use when validating a longitudinal tool's false-positive control on semi-synthetic or real count/proportion data by shuffling the continuous time variable within each individual, as done for the fasting stool microbiome benchmark.

## When NOT to use

- The data are not longitudinal or lack a within-individual time variable that can be permuted to create a valid null
- The goal is to detect real time/treatment effects rather than to benchmark false-positive control under a null hypothesis
- The analysis is on data types where LongDat's randomized control table is not produced (e.g. measurement/others data return a Normalize_method table instead)

## Inputs

- Genus-level longitudinal master table of fasting stool microbiome data (count data type) with a continuous time variable (e.g. Day) and individual identifiers
- Permutation seed (integer) for the randomized control test
- Normalization/rarefaction variants of the count feature table

## Outputs

- Per-tool, per-normalization false-positive tallies on shuffled time data
- LongDat randomized control table for count data (longdat_cont() output)
- Qualitative comparison matching Supplementary Fig. S16 (near-zero LongDat false positives; elevated rates for MaAsLin2, lgpr, ZIBR)

## How to apply

Load the genus-level longitudinal master table (e.g. the fasting stool microbiome table) and generate null datasets by permuting the time variable within each individual, which destroys true time-feature associations while preserving individual-level structure. Set a seed beforehand, because when LongDat's data_type equals 'count' the randomized control test is randomized and must be reproducible. Run longdat_cont() on each shuffled dataset with data_type = 'count' and the continuous time variable as test_var, then run the comparison tools MaAsLin2, lgpr and ZIBR on identical shuffled datasets. Repeat across the different normalization/rarefaction methods evaluated for count data, tally false positives (features called significant despite permuted time) per tool per normalization method, and check that LongDat shows few-to-no false positives while MaAsLin2, lgpr and ZIBR show measurably higher rates (Supplementary Fig. S16 pattern).

## Related tools

- **LongDat** (longdat_cont() with data_type='count' and a continuous test_var; produces the result table plus a randomized control table on shuffled (null) data) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (comparison tool run on the identical shuffled datasets to measure its false-positive rate)
- **lgpr** (comparison tool run on the identical shuffled datasets to measure its false-positive rate)
- **ZIBR** (comparison tool run on the identical shuffled datasets to measure its false-positive rate)
- **R** (implementation environment for the permutation scheme, seed setting and tool runs) — https://github.com/CCY-dev/LongDat

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- False-positive rate for LongDat on time-shuffled data is near zero in all modes (matching Supplementary Fig. S16)
- MaAsLin2, lgpr and ZIBR show measurably higher false-positive rates than LongDat on the identical shuffled datasets
- Re-running with the same pre-set seed reproduces the randomized control test exactly when data_type = 'count'
- Shuffling is performed within each individual, so individual-level structure (e.g. sample counts per subject) is preserved across permutations
- Result is consistent across the different normalization/rarefaction methods evaluated for count data

## Limitations

- When data_type equals 'count', the randomized control test is randomized, so a seed must be set for reproducibility
- Findings are qualitative for the fasting stool microbiome semi-synthetic setting; exact false-positive counts depend on the normalization/rarefaction method chosen
- The null holds only if the permutation erases all true time-feature associations; residual structure (e.g. covariate correlations with time within individuals) can distort the false-positive estimate

## Evidence

- [intro] count data requires seed for reproducible randomized control test: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] randomized control table is a third output for count data: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."
- [abstract] comparison tools benchmarked against LongDat: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
- [readme] LongDat input and purpose (README): "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at"
