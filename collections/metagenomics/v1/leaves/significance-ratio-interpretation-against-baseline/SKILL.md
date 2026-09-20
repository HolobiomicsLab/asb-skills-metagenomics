---
name: significance-ratio-interpretation-against-baseline
description: Use when evaluating whether differential abundance results at gene, KO,
  EC, or reaction level exceed what is expected by chance, to choose the annotation
  level for biomarker discovery in metagenomic case-control data.
when_to_use_negative:
- Differential abundance results were generated with a different significance threshold
  (not P ≤ 0.01), so the 1% chance baseline no longer applies
- The analysis goal is taxonomic-level (taxon) comparison rather than gene/KO/EC/reaction
  functional-level comparison
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline that produced the differential analysis results whose significance
    ratios are being benchmarked; provides gene, KO, EC and reaction level calls using
    metagenomeSeq with CSS normalization
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Execution environment for reproducing the upstream CAMAMED analysis without
    installing dependencies
  repo: https://hub.docker.com/r/camamed/camamed_pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:47631f9091fbef4e40f21cdb2a1b63d6a40cc3b41bce240f813e68629137d02f
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Significance ratio interpretation against baseline

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking interpretive skill from the CAMAMED colorectal adenoma/carcinoma test case: compare the fraction of features called significant at P ≤ 0.01 across annotation levels (gene 1.92%, KO 2.98%, EC 2.55%, reaction 2.45%) against the 1% chance baseline to justify functional-level over gene-level differential analysis.

## When to use

Use when evaluating whether differential abundance results at gene, KO, EC, or reaction level exceed what is expected by chance, to choose the annotation level for biomarker discovery in metagenomic case-control data.

## When NOT to use

- Differential abundance results were generated with a different significance threshold (not P ≤ 0.01), so the 1% chance baseline no longer applies
- The analysis goal is taxonomic-level (taxon) comparison rather than gene/KO/EC/reaction functional-level comparison

## Inputs

- Reproduced significance-ratio table from the CAMAMED colorectal adenoma/carcinoma differential analysis
- Significance threshold definition (P ≤ 0.01) used for the differential calls

## Outputs

- significance_ratio_comparison.csv with columns: annotation level, significance ratio, excess-over-baseline
- Ranking of annotation levels (gene < reaction < EC < KO) by significance ratio

## How to apply

Load the reproduced significance-ratio table from the upstream CAMAMED analysis (run via the Docker image, which requires no separate software installation). Extract the reported percentage of features detected as significant at the P ≤ 0.01 threshold for each annotation level: gene (1.92%), KO group (2.98%), EC number (2.55%) and reaction (2.45%). Set the chance baseline at 1% (a P ≤ 0.01 threshold expects ~1% false positives by chance) and compute each level's excess over this baseline. Assemble a comparison table (annotation level, significance ratio, excess-over-baseline) ordered gene < reaction < EC < KO, write it to significance_ratio_comparison.csv, and confirm the gene-level ratio sits nearest the baseline while KO/EC/reaction exceed it — the paper's rationale for preferring functional-level analysis.

## Related tools

- **CAMAMED** (Pipeline that produced the differential analysis results whose significance ratios are being benchmarked; provides gene, KO, EC and reaction level calls using metagenomeSeq with CSS normalization) — https://github.com/mhnb/camamed
- **Docker** (Execution environment for reproducing the upstream CAMAMED analysis without installing dependencies) — https://hub.docker.com/r/camamed/camamed_pipeline

## Examples

```
python camamed_init  # run inside the camamed/camamed_pipeline Docker image before computing significance_ratio_comparison.csv
```

## Evaluation signals

- Gene-level significance ratio reproduces at 1.92%, KO at 2.98%, EC at 2.55% and reaction at 2.45% at P ≤ 0.01
- Chance baseline is set to 0.01 and excess-over-baseline is non-zero and positive for KO, EC and reaction levels
- Ranking of levels in the comparison table is gene < reaction < EC < KO
- Output file significance_ratio_comparison.csv exists with the columns annotation level, significance ratio, and excess-over-baseline
- Gene-level value lies closest to the baseline while all three functional levels exceed it

## Limitations

- The reported ratios are specific to the CAMAMED colorectal adenoma/carcinoma test case at P ≤ 0.01 and may not transfer to other datasets or thresholds
- metagenomeSeq is reported to have a high false-positive rate for differential abundance analysis when the sample size is small, which can inflate significance ratios
- No software version for CAMAMED or its component tools is stated, complicating exact reproduction

## Evidence

- [results] At the same significance threshold (P ≤ 0.01), a greater ratio of KO groups, EC numbers and reactions was detected as significant (2.98%, 2.55% and 2.45%, respectively) than the 1.92% seen at the gene level: [evidence span withheld — anchor status: not_found]
