---
name: zero-inflation-correction-sparse-count-data
description: Use when analyzing sparse metagenomic taxa or gene abundance count tables
  for case-control differential abundance, where compositional bias and zero inflation
  must be corrected before calling marker taxa, genes, KOs, EC numbers, or reactions.
when_to_use_negative:
- Input is already normalized or compositional bias has been removed by another method
  (e.g., log-ratio transformed data)
- Data are not sparse metagenomic count data (e.g., RNA-seq reads where DESeq2/edgeR
  models were originally designed and are more appropriate)
- Sample size is very small, where metagenomeSeq is reported to have a high false-positive
  rate for differential abundance
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline that applies CSS normalization and metagenomeSeq differential abundance
    to taxa/gene counts; runnable via camamed_init or Docker images camamed_pipeline
    and camamed_pipeline_db
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Zero-inflated log-normal differential abundance model applied to CSS-corrected
    sparse count data to identify marker taxa and genes
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Prebuilt Docker images avoid installing dependencies when running the CSS
    + metagenomeSeq workflow
  repo: https://github.com/mhnb/camamed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:276841511e8bd2e30bb82f346afafc9274dcd9576754189e6367b3421d995686
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# zero-inflation-correction-sparse-count-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill applies cumulative sum-scaling (CSS) normalization together with metagenomeSeq's zero-inflated log-normal differential abundance model (as implemented in the CAMAMED pipeline) to correct compositional bias and zero inflation in sparse metagenomic taxa/gene count data before marker identification.

## When to use

Use when analyzing sparse metagenomic taxa or gene abundance count tables for case-control differential abundance, where compositional bias and zero inflation must be corrected before calling marker taxa, genes, KOs, EC numbers, or reactions.

## When NOT to use

- Input is already normalized or compositional bias has been removed by another method (e.g., log-ratio transformed data)
- Data are not sparse metagenomic count data (e.g., RNA-seq reads where DESeq2/edgeR models were originally designed and are more appropriate)
- Sample size is very small, where metagenomeSeq is reported to have a high false-positive rate for differential abundance

## Inputs

- Taxa abundance count table (samples x taxa) from taxonomic profiling
- Gene abundance count table (samples x genes) from mapping to a non-redundant gene catalog
- Case-control sample metadata/labels

## Outputs

- CSS-corrected normalized taxa and gene count matrices
- Differentially abundant (marker) taxa and genes with metagenomeSeq statistics (e.g., fitFeatureModel results)
- Consistency/overlap comparison of marker sets (e.g., css_cross_dataset_validation.csv)

## How to apply

Load the taxa and gene abundance count tables (e.g., from mapping metagenome sequences to non-redundant gene catalogs and MetaPhlAn2-style taxonomic profiling) and apply cumulative sum-scaling (CSS) normalization independently at the taxa and gene levels to remove compositional bias. Fit the metagenomeSeq zero-inflated log-normal model to the CSS-corrected counts to identify differentially abundant features (colorectal cancer markers) in case-control samples. Optionally repeat the differential abundance analysis without CSS to quantify how compositional bias affected the calls — in the CAMAMED evaluation, CSS-corrected markers reproduced across two independent colorectal cancer datasets while uncorrected results were inconsistent. Rationale: metagenomeSeq is chosen in CAMAMED for its capability of handling compositional bias in sparse data at taxa, gene, KO, EC number and reaction levels, and CSS handles the compositional nature of metagenomic data. Note the known caveat that metagenomeSeq has a high false-positive rate when sample size is small, so adequate sample sizes are needed.

## Related tools

- **CAMAMED** (Pipeline that applies CSS normalization and metagenomeSeq differential abundance to taxa/gene counts; runnable via camamed_init or Docker images camamed_pipeline and camamed_pipeline_db) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Zero-inflated log-normal differential abundance model applied to CSS-corrected sparse count data to identify marker taxa and genes) — https://github.com/mhnb/camamed
- **Docker** (Prebuilt Docker images avoid installing dependencies when running the CSS + metagenomeSeq workflow) — https://github.com/mhnb/camamed

## Examples

```
docker run -v $PWD:/data camamed/camamed_pipeline /bin/bash -c "python camamed_init && camamed -taxa /data/taxa_counts.csv -genes /data/gene_counts.csv --css"
```

## Evaluation signals

- CSS-corrected marker sets show substantially greater cross-dataset agreement between independent colorectal cancer datasets than uncorrected marker sets (the article reports uncorrected results were 'not consistent')
- Output consistency file (e.g., css_cross_dataset_validation.csv) exists and contains metrics for both CSS-corrected and uncorrected comparisons
- CSS normalization was applied independently at both the taxa and gene levels, not on pooled data
- Differential abundance used metagenomeSeq's zero-inflated log-normal model, not plain counts or RNA-seq-only models
- All five CAMAMED annotation levels (taxon, gene, KO, EC number, reaction) can be processed if functional tables are supplied

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance analysis when the sample size is small
- Without applying CSS, differential abundance results from two independent datasets were not consistent
- Correction for zero-inflation and compositional bias is largely neglected in other pipelines, so results are not directly comparable to uncorrected pipelines
- No software version is stated for CAMAMED or its component tools, complicating exact reproducibility

## Evidence
