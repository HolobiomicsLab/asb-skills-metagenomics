---
name: grouped-frequency-table-statistics
description: Use when you have normalized five-level frequency tables (taxon, gene,
  KO, EC number, reaction) from CAMAMED's med_pipeline and need to find features that
  differ significantly across two or more sample groups.
when_to_use_negative:
- Input data are raw counts that have not been normalized — the pipeline uses cumulative
  sum-scaling (CSS) at taxa and gene levels first because of the compositional nature
  of metagenomic data
- You only have a single group of samples — the Kruskal-Wallis H test compares a variable
  across two or more groups, so no between-group contrast exists
- You need differential abundance tools with other statistical models — the article
  notes alternatives such as metagenomeSeq (with its known high false-positive rate
  at small sample sizes), RAIDA, ANCOM, ZIBSeq, CPL, DESeq2 and edgeR
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CAMAMED
  role: Provides the statistical analysis environment in which the Kruskal-Wallis/ANOVA
    tests and Benjamini-Hochberg FDR correction are run on the five-level normalized
    frequency tables
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Execution container (camamed_pipeline / camamed_pipeline_db images) so the
    statistical step runs without installing separate software dependencies
  repo: https://hub.docker.com
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:0abe4f9ee0a1973a8f603b49dc1c8010722d564403357fc23079d3413ed5ce6b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# grouped-frequency-table-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical testing of normalized frequency tables across phenotype groups, as implemented in the CAMAMED pipeline: per-feature Kruskal-Wallis H tests (or one-way ANOVA where normality is acceptable) on healthy/adenoma/carcinoma sample groups, followed by Benjamini-Hochberg FDR correction to identify potential biomarkers in case-control metagenomic samples.

## When to use

Use when you have normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) from CAMAMED's med_pipeline and need to find features that differ significantly across two or more sample groups.

## When NOT to use

- Input data are raw counts that have not been normalized — the pipeline uses cumulative sum-scaling (CSS) at taxa and gene levels first because of the compositional nature of metagenomic data
- You only have a single group of samples — the Kruskal-Wallis H test compares a variable across two or more groups, so no between-group contrast exists
- You need differential abundance tools with other statistical models — the article notes alternatives such as metagenomeSeq (with its known high false-positive rate at small sample sizes), RAIDA, ANCOM, ZIBSeq, CPL, DESeq2 and edgeR

## Inputs

- Normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) from the CAMAMED med_pipeline
- Sample group/phenotype labels (e.g., healthy, adenoma, carcinoma)
- CAMAMED Docker image (camamed_pipeline or camamed_pipeline_db)

## Outputs

- differential_abundance_results.csv with per-feature identifier, level, test statistic, raw P-value, and Benjamini-Hochberg-adjusted P-value

## How to apply

Load the CSS-normalized frequency tables emitted by the upstream CAMAMED med_pipeline for the healthy, adenoma, and carcinoma samples. For each feature row, run the non-parametric Kruskal-Wallis H test across the three phenotype groups to compare the value of the variable, substituting one-way ANOVA where normality is judged acceptable. Record the test statistic and raw P-value for every feature, then apply the Benjamini-Hochberg procedure to the raw P-value vector to obtain FDR-adjusted P-values. Write per-feature results (feature identifier, level, statistic, raw P, BH-adjusted P) to differential_abundance_results.csv, executing inside the CAMAMED Docker image so no separate software dependencies need to be installed.

## Related tools

- **CAMAMED** (Provides the statistical analysis environment in which the Kruskal-Wallis/ANOVA tests and Benjamini-Hochberg FDR correction are run on the five-level normalized frequency tables) — https://github.com/mhnb/camamed
- **Docker** (Execution container (camamed_pipeline / camamed_pipeline_db images) so the statistical step runs without installing separate software dependencies) — https://hub.docker.com

## Examples

```
python camamed_init
```

## Evaluation signals

- differential_abundance_results.csv exists and contains the required columns: feature identifier, level, statistic, raw P, BH-adjusted P
- Every feature row in the input frequency tables has a corresponding test statistic and raw P-value (no dropped features)
- BH-adjusted P-values are monotonically consistent with the raw P-values (FDR adjustment preserves the ranking order)
- The analysis ran inside the CAMAMED Docker image, so no additional dependency installation was needed
- Tested features span all five annotation levels: taxon, gene, KO, EC number and reaction

## Limitations

- The non-parametric Kruskal-Wallis H test is the default; switching to one-way ANOVA requires a judgment that normality is acceptable
- The article reports metagenomeSeq's high false-positive rate for differential abundance at small sample sizes, a caution for grouped-testing approaches in general
- No software version is stated for CAMAMED or its component tools, and the two colorectal cancer gut metagenome datasets used are not identified by accession

## Evidence
