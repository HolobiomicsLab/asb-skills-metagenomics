---
name: kegg-ortholog-ec-reaction-enrichment-analysis
description: Use when mapping-based metagenomic count data (case-control design) has
  genes mapped to KEGG KO groups, EC numbers, and reactions, and you need to decide
  whether functional-level differential analysis outperforms gene-level testing at
  a fixed significance threshold.
when_to_use_negative:
- The goal is taxonomic profiling only — this skill addresses functional (KO/EC/reaction)
  differential analysis, not the taxon-level output of CAMAMED.
- No appropriate gene catalog is available for mapping-based analysis (the paper prefers
  mapping-based methods only 'when appropriate gene catalogs are available').
- Counts are raw/unnormalized and you intend to use conventional statistics — CSS
  normalization plus a compositional-data-aware model is required, otherwise results
  are compositional-biased.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0160
tools:
- name: CAMAMED
  role: Pipeline that maps genes to KEGG and performs composition-aware (CSS + metagenomeSeq)
    differential analysis at KO, EC number, and reaction levels
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Statistical model used by CAMAMED for differential abundance testing at taxa,
    gene, KO, EC and reaction levels
- name: Docker
  role: Docker images (camamed_pipeline / camamed_pipeline_db) provide the pre-installed
    CAMAMED environment so the chained analysis requires no separate software installation
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:84e74fcccde8268a54a4d07fa2ee6d417cf3a31c692cbd09d8dfc3012f3f6e55
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kegg-ortholog-ec-reaction-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential analysis of metagenomic data at KEGG functional levels (KO groups, EC numbers, reactions) rather than the individual gene level, quantifying the ratio of significant features at a fixed threshold (P ≤ 0.01) to show that functional annotation surfaces more signal above chance.

## When to use

Use when mapping-based metagenomic count data (case-control design) has genes mapped to KEGG KO groups, EC numbers, and reactions, and you need to decide whether functional-level differential analysis outperforms gene-level testing at a fixed significance threshold.

## When NOT to use

- The goal is taxonomic profiling only — this skill addresses functional (KO/EC/reaction) differential analysis, not the taxon-level output of CAMAMED.
- No appropriate gene catalog is available for mapping-based analysis (the paper prefers mapping-based methods only 'when appropriate gene catalogs are available').
- Counts are raw/unnormalized and you intend to use conventional statistics — CSS normalization plus a compositional-data-aware model is required, otherwise results are compositional-biased.

## Inputs

- Metagenomic read sets from case-control samples (e.g. colorectal adenoma/carcinoma gut metagenomes)
- Non-redundant gene catalog for mapping-based gene frequency estimation
- KEGG database mapping of genes to KO groups, EC numbers, and reactions
- Case-control sample metadata (grouping variable for metagenomeSeq)

## Outputs

- Per-level significance ratios (fraction of features significant at P ≤ 0.01) for gene, KO, EC number, and reaction levels
- significance_ratio_comparison.csv with columns: annotation level, significance ratio, excess-over-baseline
- Set of significant KO groups, EC numbers, and reactions as candidate functional biomarkers

## How to apply

Run the CAMAMED pipeline (e.g. via its Docker images so no separate dependencies are installed) to map metagenome reads to a non-redundant gene catalog, then map genes to the KEGG database to derive KO, EC, and reaction annotations. Apply CSS normalization and metagenomeSeq's zero-inflated model to test each feature for differential abundance between case and control samples. At the same threshold (P ≤ 0.01), compute the fraction of significant features at each level; in the CAMAMED colorectal adenoma/carcinoma test case this yielded KO = 2.98%, EC = 2.55%, reaction = 2.45% versus 1.92% at the gene level. Tabulate level, significance ratio, and excess over the 0.01 chance baseline (ordering gene < reaction < EC < KO) and write the comparison to significance_ratio_comparison.csv; the gene-level value should sit near the baseline while KO/EC/reaction values exceed it, supporting the paper's recommendation to prefer functional-level analysis.

## Related tools

- **CAMAMED** (Pipeline that maps genes to KEGG and performs composition-aware (CSS + metagenomeSeq) differential analysis at KO, EC number, and reaction levels) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Statistical model used by CAMAMED for differential abundance testing at taxa, gene, KO, EC and reaction levels)
- **Docker** (Docker images (camamed_pipeline / camamed_pipeline_db) provide the pre-installed CAMAMED environment so the chained analysis requires no separate software installation) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
python camamed_init
```

## Evaluation signals

- At P ≤ 0.01, the gene-level significance ratio reproduces to ~1.92%, close to the 0.01 chance baseline.
- KO, EC, and reaction ratios reproduce to ~2.98%, 2.55%, and 2.45% respectively, each exceeding the baseline and satisfying the ordering gene < reaction < EC < KO.
- significance_ratio_comparison.csv exists with the three expected columns (annotation level, significance ratio, excess-over-baseline).
- Excess-over-baseline values for KO/EC/reaction are all positive (≈ 0.0198, 0.0155, 0.0145).
- All five annotation levels (taxon, gene, KO, EC number, reaction) have been run through CSS normalization before testing.

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance when the sample size is small.
- Without applying CSS, results from the two colorectal cancer gut metagenome datasets were not consistent.
- The two colorectal cancer gut metagenome datasets used for validation are not identified by accession, limiting direct reuse of the benchmark.
- No software version is stated for CAMAMED or its component tools, so reproducibility relies on the Docker image.

## Evidence

- [results] KO/EC/reaction significance ratios exceed gene level: [evidence span withheld — anchor status: not_found]
