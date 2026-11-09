---
name: metagenomic-functional-annotation-level-comparison
description: Decision support for choosing between gene-level and functional-level
  (KO/EC/reaction) differential analysis of mapped metagenomic count data, when you
  have per-level significance ratios from a case-control dataset such as CAMAMED's
  colorectal adenoma/carcinoma (CAMAMED) run.
when_to_use_negative:
- Input is already a functional-level (KO/EC/reaction) feature table and the question
  is biomarker identification rather than level comparison — use CAMAMED's functional
  profiling workflow instead.
- 'The data have not been normalized for compositional bias: raw mapped counts without
  CSS normalization were shown by the paper to give inconsistent results across the
  two colorectal cancer datasets.'
- Small sample sizes where metagenomeSeq alone has a reported high false-positive
  rate for differential abundance analysis.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: CAMAMED
  role: Pipeline that produces the per-level significance ratios via metagenomeSeq
    with CSS normalization; runs in the provided Docker image so no separate installation
    is needed
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Runtime environment for reproducing the upstream CAMAMED analysis whose significance-ratio
    table feeds this comparison
- name: metagenomeSeq
  role: Statistical model used within CAMAMED for differential abundance at taxa,
    gene, KO, EC number and reaction levels
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:54313b9345f264d0fbb79abb561c497b15708bea1e1c5b47cf1bd11d06287ca1
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metagenomic-functional-annotation-level-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares differential-abundance significance ratios across annotation levels (gene, KEGG ortholog (KO), EC number, reaction) to decide whether functional-level analysis outperforms gene-level analysis. In the CAMAMED colorectal adenoma/carcinoma test case, KO/EC/reaction levels exceeded the gene-level significance rate at the same P ≤ 0.01 threshold.

## When to use

Decision support for choosing between gene-level and functional-level (KO/EC/reaction) differential analysis of mapped metagenomic count data, when you have per-level significance ratios from a case-control dataset such as CAMAMED's colorectal adenoma/carcinoma (CAMAMED) run.

## When NOT to use

- Input is already a functional-level (KO/EC/reaction) feature table and the question is biomarker identification rather than level comparison — use CAMAMED's functional profiling workflow instead.
- The data have not been normalized for compositional bias: raw mapped counts without CSS normalization were shown by the paper to give inconsistent results across the two colorectal cancer datasets.
- Small sample sizes where metagenomeSeq alone has a reported high false-positive rate for differential abundance analysis.

## Inputs

- Significance-ratio table from CAMAMED differential analysis (metagenomeSeq with CSS normalization) of the CAMAMED colorectal adenoma/carcinoma datasets, per annotation level (gene, KO, EC number, reaction)
- Significance threshold (P ≤ 0.01) used to define detected features per level

## Outputs

- significance_ratio_comparison.csv with columns: annotation level, significance ratio, excess-over-baseline
- Ordered comparison (gene < reaction < EC < KO) showing which annotation levels exceed the chance baseline

## How to apply

Load the reproduced significance-ratio table from the upstream CAMAMED Docker-image analysis of the CAMAMED colorectal adenoma/carcinoma data. Extract the reported significance ratios at the P ≤ 0.01 threshold: gene (1.92%), KO (2.98%), EC number (2.55%), and reaction (2.45%). Set the chance/significance baseline at 0.01 (the 1% expected under the threshold) and compute each level's excess over baseline. Assemble a comparison table with columns for annotation level, significance ratio, and excess-over-baseline, ordered gene < reaction < EC < KO, and write it to significance_ratio_comparison.csv. Confirm the gene-level value lies near the baseline while KO, EC, and reaction values exceed it — this supports the paper's recommendation to analyze at KO/EC/reaction levels rather than the gene level.

## Related tools

- **CAMAMED** (Pipeline that produces the per-level significance ratios via metagenomeSeq with CSS normalization; runs in the provided Docker image so no separate installation is needed) — https://github.com/mhnb/camamed
- **Docker** (Runtime environment for reproducing the upstream CAMAMED analysis whose significance-ratio table feeds this comparison)
- **metagenomeSeq** (Statistical model used within CAMAMED for differential abundance at taxa, gene, KO, EC number and reaction levels)

## Examples

```
python camamed_init
```

## Evaluation signals

- Gene-level significance ratio equals 1.92% at P ≤ 0.01, matching the paper's reported value.
- KO, EC, and reaction ratios equal 2.98%, 2.55%, and 2.45% respectively at P ≤ 0.01, all exceeding the gene-level value.
- All levels' excess-over-baseline is computed against the 0.01 chance rate; gene-level excess is smallest and KO excess is largest.
- Comparison table written to significance_ratio_comparison.csv with columns annotation level, significance ratio, excess-over-baseline, ordered gene < reaction < EC < KO.
- Concluded ordering is consistent with the paper's finding that functional analysis at KO/EC/reaction levels outperforms gene-level analysis.

## Limitations

- The two colorectal cancer gut metagenome datasets are not identified by accession, hindering exact reproduction from the paper alone.
- Without applying CSS normalization, results from the two colorectal cancer datasets were not consistent, so the comparison depends on the compositional preprocessing step.
- metagenomeSeq has a reported high false-positive rate for differential abundance analysis when sample size is small.
- No software version is stated for CAMAMED or its component tools.

## Evidence

- [results] At the same significance threshold (P ≤ 0.01), a greater ratio of KO groups, EC numbers and reactions was detected as significant (2.98%, 2.55% and 2.45%, respectively) than the 1.92% seen at the gene level, supporting functional analysis at KO/EC/reaction levels rather than the gene level.: [evidence span withheld — anchor status: not_found]
