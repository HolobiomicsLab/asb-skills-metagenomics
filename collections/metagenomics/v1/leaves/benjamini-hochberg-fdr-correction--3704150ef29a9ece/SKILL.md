---
name: benjamini-hochberg-fdr-correction
description: Use when comparing case-control metagenomic feature tables (e.g. healthy
  vs adenoma vs carcinoma CSS-normalized abundance at five annotation levels) and
  you need per-feature P-values corrected for multiple testing.
when_to_use_negative:
- Input data are raw un-normalized gene counts — compositional bias must first be
  removed with CSS normalization before testing
- The goal is a single pre-specified comparison with no multiple-testing burden across
  thousands of features, where BH correction is unnecessary
- Normality of the feature distribution is acceptable and the analysis plan calls
  for one-way ANOVA instead of the non-parametric test
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3055
tools:
- name: CAMAMED
  role: Pipeline providing the Kruskal-Wallis H test with Benjamini-Hochberg FDR correction
    over the five-level normalized frequency tables in its med_pipeline environment
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Execution environment (camamed_pipeline / camamed_pipeline_db images) so the
    statistical testing stage runs without installing dependencies separately
  repo: https://hub.docker.com/r/camamed/camamed_pipeline
- name: metagenomeSeq
  role: Upstream tool used by CAMAMED for CSS normalization of sparse taxa and gene
    abundance data feeding the differential testing stage; noted for high false-positive
    rate at small sample sizes, motivating the pipeline's design
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:035b0554ef9063c032e1f558a0f3345f79e29d4aa1b5de2b2bd0d4a71467fed5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3055
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# benjamini-hochberg-fdr-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical significance testing stage of the CAMAMED metagenomic pipeline: per-feature non-parametric testing across phenotype groups (Kruskal-Wallis H test, with one-way ANOVA where normality is acceptable) followed by Benjamini-Hochberg FDR correction of raw P-values across taxon, gene, KO, EC number and reaction levels.

## When to use

Use when comparing case-control metagenomic feature tables (e.g. healthy vs adenoma vs carcinoma CSS-normalized abundance at five annotation levels) and you need per-feature P-values corrected for multiple testing.

## When NOT to use

- Input data are raw un-normalized gene counts — compositional bias must first be removed with CSS normalization before testing
- The goal is a single pre-specified comparison with no multiple-testing burden across thousands of features, where BH correction is unnecessary
- Normality of the feature distribution is acceptable and the analysis plan calls for one-way ANOVA instead of the non-parametric test

## Inputs

- CSS-normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) from the CAMAMED med_pipeline
- Sample-to-phenotype grouping metadata (e.g. healthy, adenoma, carcinoma)
- Significance threshold (e.g. P <= 0.01)

## Outputs

- differential_abundance_results.csv with per-feature identifier, level, test statistic, raw P-value and BH-adjusted P-value
- Per-level fractions of significantly changed entities (e.g. species 2.67%, genes 1.92%, KO 2.98%, EC 2.55%, reactions 2.45% at P <= 0.01)

## How to apply

Load the CSS-normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) emitted by the CAMAMED med_pipeline. For each feature row, run the Kruskal-Wallis H test across the phenotype groups (healthy, adenoma, carcinoma); where normality is judged acceptable, substitute a one-way ANOVA, since Kruskal-Wallis is the non-parametric option that compares a variable across two or more groups. Record each feature's test statistic and raw P-value, then apply the Benjamini-Hochberg procedure to the raw P-value vector to obtain FDR-adjusted P-values, thresholding at the significance cutoff of the analysis (the CAMAMED benchmark used P <= 0.01). Write per-feature results (feature identifier, level, statistic, raw P, BH-adjusted P) to differential_abundance_results.csv, executing inside the CAMAMED Docker image so no dependencies are installed separately.

## Related tools

- **CAMAMED** (Pipeline providing the Kruskal-Wallis H test with Benjamini-Hochberg FDR correction over the five-level normalized frequency tables in its med_pipeline environment) — https://github.com/mhnb/camamed
- **Docker** (Execution environment (camamed_pipeline / camamed_pipeline_db images) so the statistical testing stage runs without installing dependencies separately) — https://hub.docker.com/r/camamed/camamed_pipeline
- **metagenomeSeq** (Upstream tool used by CAMAMED for CSS normalization of sparse taxa and gene abundance data feeding the differential testing stage; noted for high false-positive rate at small sample sizes, motivating the pipeline's design)

## Examples

```
python camamed_init
```

## Evaluation signals

- differential_abundance_results.csv contains a row for every feature at all five levels with columns for feature identifier, level, statistic, raw P and BH-adjusted P
- Adjusted P-values are monotonically consistent with raw P-values (larger raw P never yields smaller BH-adjusted P within a level)
- Reproduced significant-entity ratios at P <= 0.01 match the reported benchmark: species 2.67%, genes 1.92%, KO groups 2.98%, EC numbers 2.55%, reactions 2.45%
- The mapped-gene count in the underlying cohort analysis equals the expected 3,354,281 genes before testing
- A greater ratio of KO groups, EC numbers and reactions than genes is flagged as significantly changed, as reported in the Table 2 benchmark

## Limitations

- metagenomeSeq, used upstream for the normalization feeding these tests, has a high false-positive rate for differential abundance analysis when the sample size is small
- Without applying CSS normalization, results from the two colorectal cancer datasets were not consistent, so testing on un-normalized tables is unreliable
- No software version is stated for CAMAMED or its component tools, complicating exact reproduction of the testing environment

## Evidence

- [other] CAMAMED identifies components that are changed significantly in different sample groups using the non-parametric Kruskal-Wallis H test, which compares the value of a variable across two or more groups.: [evidence span withheld — anchor status: not_found]
- [abstract] The pipeline supports biomarker discovery in case-control samples via functional difference testing across annotation levels.: "the pipeline enables the user to identify potential biomarkers in case-control metagenomic samples by investigating functional differences"
- [intro] metagenomeSeq's small-sample false-positive rate motivates the pipeline's statistical choices.: "one of the most important drawbacks reported for metagenomeSeq in different studies is the high false-positive rate for differential abundance analysis when the sample size is small"
- [readme] The pipeline runs from ready-made Docker images without separate dependency installation.: "Note: There is no need to install any software dependencies if you use the Docker image."
