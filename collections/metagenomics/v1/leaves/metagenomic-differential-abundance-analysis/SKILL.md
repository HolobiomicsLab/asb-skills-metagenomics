---
name: metagenomic-differential-abundance-analysis
description: Use when comparing metagenomic abundance profiles (taxon/gene/KO/EC/reaction
  tables) across phenotype groups (e.g. healthy vs adenoma vs carcinoma) to find significantly
  changed biomarker features while correcting compositional bias.
when_to_use_negative:
- Input is raw metagenomic reads rather than normalized abundance/feature tables —
  run the upstream CAMAMED mapping and CSS normalization first
- The data are not compositional count/frequency data (e.g. RNA-seq with DESeq2/edgeR
  is a different context; the paper notes these were originally developed for RNA-seq)
- Sample size is small and false-positive control is critical — the paper reports
  metagenomeSeq has a high false-positive rate for differential abundance when sample
  size is small
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline providing normalized five-level frequency tables and the med_pipeline
    environment for running Kruskal-Wallis/BH differential testing
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Zero-inflated log-normal differential abundance model fit to CSS-normalized
    taxa and gene tables
- name: Docker
  role: Execution environment (camamed_pipeline / camamed_pipeline_db images) so no
    separate software dependencies need to be installed
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:f195f23f33c3a555236075bb08ffc6fd85d3b59a7d9a03b1d94e4b7e0a235182
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
  - BiocManager
  - CAMAMED
  - Docker
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metagenomic-differential-abundance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Composition-aware identification of taxa, genes, KOs, EC numbers, and reactions that differ significantly between case-control metagenomic sample groups, using CAMAMED's CSS-normalized metagenomeSeq modeling plus Kruskal-Wallis H testing with Benjamini-Hochberg FDR correction.

## When to use

Use when comparing metagenomic abundance profiles (taxon/gene/KO/EC/reaction tables) across phenotype groups (e.g. healthy vs adenoma vs carcinoma) to find significantly changed biomarker features while correcting compositional bias.

## When NOT to use

- Input is raw metagenomic reads rather than normalized abundance/feature tables — run the upstream CAMAMED mapping and CSS normalization first
- The data are not compositional count/frequency data (e.g. RNA-seq with DESeq2/edgeR is a different context; the paper notes these were originally developed for RNA-seq)
- Sample size is small and false-positive control is critical — the paper reports metagenomeSeq has a high false-positive rate for differential abundance when sample size is small

## Inputs

- Normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) from the CAMAMED med_pipeline
- Sample phenotype group assignments (e.g. healthy, adenoma, carcinoma)
- Taxa and gene abundance count tables from independent gut metagenome datasets

## Outputs

- differential_abundance_results.csv (feature identifier, level, Kruskal-Wallis statistic, raw P, BH-adjusted P)
- css_cross_dataset_validation.csv (consistency metrics for CSS-corrected vs uncorrected marker sets across datasets)
- Colorectal cancer marker taxa and gene lists per dataset

## How to apply

Load the normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) emitted by the CAMAMED med_pipeline. Apply cumulative sum-scaling (CSS) normalization at the taxa and gene levels, then fit the metagenomeSeq zero-inflated log-normal differential abundance model, which CAMAMED uses for its high capability of handling compositional bias in sparse metagenomic data. Independently, for each feature row, run the non-parametric Kruskal-Wallis H test across the sample groups (substituting one-way ANOVA where normality is judged acceptable) and record test statistics and raw P-values. Apply the Benjamini-Hochberg procedure to the raw P-value vector to obtain FDR-adjusted P-values. Execute inside the CAMAMED Docker image (camamed_pipeline or camamed_pipeline_db) so no separate software dependencies need to be installed, and validate marker reproducibility on a second independent dataset.

## Related tools

- **CAMAMED** (Pipeline providing normalized five-level frequency tables and the med_pipeline environment for running Kruskal-Wallis/BH differential testing) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Zero-inflated log-normal differential abundance model fit to CSS-normalized taxa and gene tables)
- **Docker** (Execution environment (camamed_pipeline / camamed_pipeline_db images) so no separate software dependencies need to be installed) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
docker run -v $PWD:/data camamed/camamed_pipeline_db python camamed_init  # then run Kruskal-Wallis + BH FDR via med_pipeline on the five-level frequency tables
```

## Evaluation signals

- Per-feature output table contains feature identifier, level, test statistic, raw P-value, and BH-adjusted P-value for every row
- Adjusted P-values from the Benjamini-Hochberg procedure are monotone with and no smaller than the raw P-values
- CSS-corrected marker sets reproduce across two independent colorectal cancer datasets, while uncorrected marker sets do not (paper: 'without applying the CSS, the results from these datasets were not consistent')
- Output file css_cross_dataset_validation.csv exists and contains consistency metrics for both CSS-corrected and uncorrected comparisons
- The analysis runs inside the CAMAMED Docker image without requiring additional software installation

## Limitations

- metagenomeSeq has a high false-positive rate for differential abundance analysis when the sample size is small
- Without applying the CSS compositional correction, results across datasets are not consistent
- No software version is stated for CAMAMED or its component tools
- The two colorectal cancer datasets are not identified by accession, complicating direct reproduction

## Evidence
