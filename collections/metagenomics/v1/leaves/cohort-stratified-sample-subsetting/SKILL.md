---
name: cohort-stratified-sample-subsetting
description: Use when you have sample-level metadata (continent, country, sample source,
  disease status) plus paired fungal and bacterial taxonomic profiles and need to
  isolate a specific cohort before per-cohort modeling or network analysis.
when_to_use_negative:
- The input table already contains only a single pre-defined cohort and no further
  stratification is needed
- The metadata lacks any of the stratification fields (continent, country, sample
  source, disease status), making cohort definition ambiguous
- The goal is a pan-cohort (all 2,679 samples) analysis rather than a cohort-specific
  comparison
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: FunOMIC
  role: Produces the fungal taxonomic profiles that are subset per cohort via the
    metadata fields
  repo: https://github.com/ManichanhLab/FunOMIC
- name: pandas
  role: Python module (required by the FunOMIC pipeline) used for metadata-filtered
    subsetting of profile tables
- name: SparCC
  role: Downstream consumer of the subsetted cohort tables; run separately per cohort
    to estimate inter-kingdom correlations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:735bfcd0fa69a6a9c543e9d3de802b6646df5327b359ffc87992a0d0e1c7e7ba
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - FunOMIC
  - MetaPhlAn
  - SparCC
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cohort-stratified-sample-subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies the 2,679-sample human metagenome collection into well-defined clinical/demographic cohorts (e.g. healthy Danish gut, healthy Spanish gut) by filtering sample metadata, so that downstream per-cohort analyses such as SparCC inter-kingdom correlation are run on matched fungal (FunOMIC-T) and bacterial (MetaPhlAn) profile tables.

## When to use

Use when you have sample-level metadata (continent, country, sample source, disease status) plus paired fungal and bacterial taxonomic profiles and need to isolate a specific cohort before per-cohort modeling or network analysis.

## When NOT to use

- The input table already contains only a single pre-defined cohort and no further stratification is needed
- The metadata lacks any of the stratification fields (continent, country, sample source, disease status), making cohort definition ambiguous
- The goal is a pan-cohort (all 2,679 samples) analysis rather than a cohort-specific comparison

## Inputs

- Fungal taxonomic profile table (samples x taxa) from FunOMIC-T mapping of 2,679 human metagenomes
- Bacterial taxonomic profile table from MetaPhlAn v3.0.9
- Sample metadata table with fields: continent, country, sample source (body site), disease status

## Outputs

- Per-cohort fungal taxonomic profile table (e.g. healthy Danish gut, healthy Spanish gut)
- Per-cohort bacterial taxonomic profile table with identical sample IDs
- Cohort sample-ID lists / subset summary per (country, body site, disease status) combination

## How to apply

Load the sample metadata alongside the fungal taxonomic profiles (from mapping reads to the FunOMIC-T database with bowtie2) and the bacterial profiles (MetaPhlAn v3.0.9). Apply metadata-based boolean filters on the fields continent, country, sample source, and disease status — e.g. select samples where country is Denmark or Spain, sample source is gut, and disease status is healthy. Subset both the fungal and bacterial profile tables with the identical retained sample-ID list so the two tables stay row-aligned for downstream joint analyses. Rationale: the paper's inter-kingdom correlation results are cohort-specific (4,184 significant SparCC correlations in the Danish healthy gut cohort vs 3,471 in the Spanish cohort), so mis-scoped subsetting would conflate populations and change the correlation counts. Verify the subset by checking that sample IDs in both profile tables match exactly and that per-cohort SparCC edge counts can be reproduced against the reported values.

## Related tools

- **FunOMIC** (Produces the fungal taxonomic profiles that are subset per cohort via the metadata fields) — https://github.com/ManichanhLab/FunOMIC
- **pandas** (Python module (required by the FunOMIC pipeline) used for metadata-filtered subsetting of profile tables)
- **SparCC** (Downstream consumer of the subsetted cohort tables; run separately per cohort to estimate inter-kingdom correlations)

## Examples

```
keep = meta[(meta.country=='Denmark') & (meta.source=='gut') & (meta.disease_status=='healthy')].sample_id; fungal_dk = fungal.loc[keep]; bacterial_dk = bacterial.loc[keep]
```

## Evaluation signals

- Fungal and bacterial subset tables contain exactly the same set of sample IDs after filtering
- No samples with non-healthy disease status or non-gut sample source remain in the 'healthy gut' subsets
- Per-cohort SparCC runs on the subsets reproduce the reported significant-edge counts (4,184 Danish; 3,471 Spanish under p < 0.05; 4,184/3,471 retained under p < 0.001 and |rho| > 0.05 criteria as reported)
- Subset sample counts are a strict subset of the full 2,679-sample set and sum across cohorts without overlap for disjoint filter criteria

## Limitations

- Cohort assignment depends entirely on the completeness and correctness of the metadata fields; missing or inconsistent disease-status labels will misassign samples
- Imbalanced cohort sizes (Danish vs Spanish) mean per-cohort correlation counts are not directly comparable without accounting for sample numbers
- FunOMIC extraction and quality of fungal marker genes rely on genome completeness, so cohort-level fungal coverage may be lower than bacterial coverage in the same subsets

## Evidence

- [other] Subset both profile tables to the healthy Danish and healthy Spanish gut cohorts using the metadata fields (continent, country, sample source, disease status).: [evidence span withheld — anchor status: not_found]
