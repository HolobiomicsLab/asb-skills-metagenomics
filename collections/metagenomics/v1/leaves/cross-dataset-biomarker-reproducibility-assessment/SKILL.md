---
name: cross-dataset-biomarker-reproducibility-assessment
description: Use when a biomarker-discovery pipeline (e.g. CAMAMED with cumulative
  sum-scaling and metagenomeSeq) has produced case-control marker taxa/genes from
  one metagenome dataset and an independent dataset from the same study context is
  available to test whether the markers reproduce.
when_to_use_negative:
- The analysis goal is taxonomic/functional profiling itself rather than validating
  whether discovered biomarkers reproduce across datasets
- Only a single dataset is available — no independent second cohort exists to test
  cross-dataset marker consistency
- The count tables are already normalized or derived from RNA-seq-style workflows;
  metagenomeSeq's zero-inflated log-normal model with CSS is designed for sparse metagenomic
  count data and CAMAMED's compositional correction would not be appropriate
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: CAMAMED
  role: Pipeline providing the CSS compositional correction and differential abundance
    analysis of taxa and gene abundance tables
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Zero-inflated log-normal differential abundance model used with CSS normalization
    on taxa, gene, KO, EC number and reaction levels
- name: Docker
  role: Docker image (camamed_pipeline / camamed_pipeline_db) to run the pipeline
    without installing dependencies separately
  repo: https://hub.docker.com/r/camamed/camamed_pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:64e696defaff4814445d410e054c338fcc594be9c2aad7b806105f1a4fe1ff25
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-dataset-biomarker-reproducibility-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates metagenomic case-control biomarker discoveries by rerunning the CSS-corrected metagenomeSeq differential abundance analysis on a second, independent dataset (taxon and gene levels) and comparing marker-set overlap against uncorrected analyses, which confirms that compositional correction — not dataset-specific noise — drives the findings.

## When to use

Use when a biomarker-discovery pipeline (e.g. CAMAMED with cumulative sum-scaling and metagenomeSeq) has produced case-control marker taxa/genes from one metagenome dataset and an independent dataset from the same study context is available to test whether the markers reproduce.

## When NOT to use

- The analysis goal is taxonomic/functional profiling itself rather than validating whether discovered biomarkers reproduce across datasets
- Only a single dataset is available — no independent second cohort exists to test cross-dataset marker consistency
- The count tables are already normalized or derived from RNA-seq-style workflows; metagenomeSeq's zero-inflated log-normal model with CSS is designed for sparse metagenomic count data and CAMAMED's compositional correction would not be appropriate

## Inputs

- Taxa abundance count table from the first colorectal cancer gut metagenome dataset
- Gene abundance count table from the first colorectal cancer gut metagenome dataset
- Taxa abundance count table from the second, independent colorectal cancer gut metagenome dataset
- Gene abundance count table from the second, independent colorectal cancer gut metagenome dataset

## Outputs

- CSS-corrected differential abundance marker taxa and genes per dataset
- Uncorrected (no-CSS) differential abundance marker taxa and genes per dataset
- Cross-dataset consistency summary table (css_cross_dataset_validation.csv) with overlap metrics for corrected vs uncorrected marker sets

## How to apply

Load the taxa and gene abundance count tables extracted from the two independent colorectal cancer gut metagenome datasets. For each dataset independently, apply cumulative sum-scaling (CSS) normalization at the taxa and gene levels and fit the metagenomeSeq zero-inflated log-normal differential abundance model to extract colorectal cancer marker taxa and genes. Repeat the identical analysis WITHOUT CSS correction on both datasets to obtain the uncorrected marker sets as a negative control. Compare marker sets across the two datasets, quantifying overlap/consistency of CSS-corrected versus uncorrected markers; the rationale is that CAMAMED's CSS step corrects zero-inflation and compositional bias, so corrected markers should agree across datasets while uncorrected ones should not. Write the comparison summary to a validation table (e.g. css_cross_dataset_validation.csv) containing consistency metrics for both corrected and uncorrected comparisons.

## Related tools

- **CAMAMED** (Pipeline providing the CSS compositional correction and differential abundance analysis of taxa and gene abundance tables) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Zero-inflated log-normal differential abundance model used with CSS normalization on taxa, gene, KO, EC number and reaction levels)
- **Docker** (Docker image (camamed_pipeline / camamed_pipeline_db) to run the pipeline without installing dependencies separately) — https://hub.docker.com/r/camamed/camamed_pipeline

## Examples

```
python camamed_init
```

## Evaluation signals

- The CSS-corrected marker sets from the two independent datasets show substantially greater cross-dataset agreement than the uncorrected marker sets
- All markers reported from the first dataset are reproduced when CAMAMED with CSS is applied to the second dataset
- Without CSS, marker sets from the two datasets are inconsistent — this expected divergence confirms the comparison was performed on the right pairs
- The output file css_cross_dataset_validation.csv exists and contains consistency metrics for both CSS-corrected and uncorrected comparisons
- CSS normalization was applied independently to each dataset at both taxa and gene levels (not pooled across datasets)

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance analysis when the sample size is small
- The two colorectal cancer gut metagenome datasets are not identified by accession, so the exact input data cannot be retrieved from the article metadata alone
- No software version is stated for CAMAMED or its component tools, which complicates exact reproducibility

## Evidence

- [results] Applying CAMAMED to another independent dataset from ref (30) reproduced all reported results: [evidence span withheld — anchor status: not_found]
