---
name: kruskal-wallis-multi-group-testing
description: Use when normalized five-level frequency tables (taxon, gene, KO, EC
  number, reaction) from case-control metagenomic samples (e.g. healthy/adenoma/carcinoma)
  need per-feature significance testing across two or more groups.
when_to_use_negative:
- The input is raw metagenomic reads or unmapped counts — CSS normalization via the
  upstream CAMAMED med_pipeline stage must run first
- Only a single sample group is available, since the Kruskal-Wallis H test compares
  a variable across two or more groups
- The goal is taxonomic/functional profiling or annotation rather than differential
  testing between groups — use the upstream profiling stages instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CAMAMED
  role: Pipeline whose med_pipeline environment provides the statistical routine for
    the Kruskal-Wallis H test, optional one-way ANOVA, and Benjamini-Hochberg FDR
    correction
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Execution environment (camamed_pipeline / camamed_pipeline_db images) so no
    statistical dependencies need to be installed separately
  repo: https://hub.docker.com
- name: metagenomeSeq
  role: Upstream compositional bias handling (with CSS normalization) producing the
    sparse taxa/gene/KO/EC/reaction data that this testing stage consumes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:d2e6d68b2c4e90c2218ecf2a6c6f5fd0ba0ee14a8ec8c5e164be7d319b031e16
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# kruskal-wallis-multi-group-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CAMAMED's statistical analysis stage: detect metagenomic components (taxon, gene, KO, EC number, reaction) that change significantly across case-control sample groups by running the non-parametric Kruskal-Wallis H test per feature, then applying Benjamini-Hochberg FDR correction to the raw P-values.

## When to use

Use when normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) from case-control metagenomic samples (e.g. healthy/adenoma/carcinoma) need per-feature significance testing across two or more groups.

## When NOT to use

- The input is raw metagenomic reads or unmapped counts — CSS normalization via the upstream CAMAMED med_pipeline stage must run first
- Only a single sample group is available, since the Kruskal-Wallis H test compares a variable across two or more groups
- The goal is taxonomic/functional profiling or annotation rather than differential testing between groups — use the upstream profiling stages instead

## Inputs

- Normalized (CSS-processed) five-level frequency tables from CAMAMED med_pipeline: taxon, gene, KO, EC number, and reaction level abundance per sample
- Sample-to-phenotype-group mapping for the case-control comparison (e.g. healthy, adenoma, carcinoma)

## Outputs

- differential_abundance_results.csv containing per-feature identifier, level, test statistic, raw P-value, and Benjamini-Hochberg FDR-adjusted P-value

## How to apply

Load the CSS-normalized frequency tables emitted by the upstream CAMAMED med_pipeline for each phenotype group (e.g. healthy, adenoma, carcinoma). For every feature row run the non-parametric Kruskal-Wallis H test across the groups; when normality is judged acceptable, substitute a one-way ANOVA. Record each feature's test statistic and raw P-value, then apply the Benjamini-Hochberg procedure to the raw P-value vector to obtain FDR-adjusted P-values. Write per-feature results (feature identifier, level, statistic, raw P, BH-adjusted P) to differential_abundance_results.csv. Execute inside the CAMAMED Docker image (camamed_pipeline or camamed_pipeline_db) so no separate dependencies must be installed; a key rationale is that metagenomeSeq alone has a high false-positive rate at small sample sizes, so rigorous per-feature testing on CSS-normalized compositional data is needed to identify biomarkers reliably.

## Related tools

- **CAMAMED** (Pipeline whose med_pipeline environment provides the statistical routine for the Kruskal-Wallis H test, optional one-way ANOVA, and Benjamini-Hochberg FDR correction) — https://github.com/mhnb/camamed
- **Docker** (Execution environment (camamed_pipeline / camamed_pipeline_db images) so no statistical dependencies need to be installed separately) — https://hub.docker.com
- **metagenomeSeq** (Upstream compositional bias handling (with CSS normalization) producing the sparse taxa/gene/KO/EC/reaction data that this testing stage consumes)

## Examples

```
python camamed_init
```

## Evaluation signals

- differential_abundance_results.csv exists and contains one row per input feature with columns for feature identifier, level, statistic, raw P, and BH-adjusted P
- The BH-adjusted P-value vector is monotone-consistent with the raw P-values (FDR >= raw P for every feature) and bounded in [0, 1]
- Features flagged as significantly changed match known biomarker-level differences between the healthy/adenoma/carcinoma groups, consistent with the pipeline's stated biomarker-discovery purpose
- Results are reproduced when re-run inside the camamed_pipeline Docker image, indicating no dependency on the host environment
- Features from all five levels (taxon, gene, KO, EC number, reaction) are represented if their upstream tables were provided

## Limitations

- No software version is stated for CAMAMED or its component tools, so exact statistical routine implementations may vary between releases
- The article does not name or accession the two colorectal cancer gut metagenome datasets used, limiting direct reproduction against the published data
- The choice between Kruskal-Wallis H and one-way ANOVA depends on a normality judgment that is described qualitatively ('where normality is judged acceptable') without a stated threshold
- The source card workflow is a reconstruction; the article's indexed findings emphasize CSS normalization and metagenomeSeq handling of compositional bias, not the testing procedure itself

## Evidence
