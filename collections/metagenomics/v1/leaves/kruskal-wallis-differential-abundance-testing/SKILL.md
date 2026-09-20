---
name: kruskal-wallis-differential-abundance-testing
description: Use when testing case-control shotgun metagenome samples (>=3 groups)
  for significantly changed taxa or functional features after CSS normalization, e.g.
  reporting ratios of changed KO/EC/reaction entities at P <= 0.01.
when_to_use_negative:
- Input is already a normalized feature table or compositional bias has not been removed
  with CSS — the article reports that without CSS, results from the two colorectal
  cancer datasets were not consistent
- Only two sample groups and a large sample size where metagenomeSeq's known high
  false-positive rate is not the target trade-off, or where an assembly-based functional
  profile is required instead of mapping to a gene catalog
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline that performs the differential abundance testing at taxon, gene,
    KO, EC number and reaction levels after CSS normalization
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Zero-inflated log-normal model engine used by CAMAMED for differential abundance
    on CSS-normalized data
- name: Docker
  role: Ready-to-use Docker images (camamed_pipeline, camamed_pipeline_db) so no dependencies
    need to be installed separately
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:525adcc6fb9a13684d444155be31787bebd3db2f73ab46df7af53150aaa65dc2
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# kruskal-wallis-differential-abundance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential abundance testing across multiple disease groups (healthy, adenoma, carcinoma) using the Kruskal-Wallis H test with Benjamini-Hochberg correction, as implemented in the CAMAMED pipeline on CSS-normalized taxa and gene abundance tables at taxon, gene, KO, EC number and reaction levels.

## When to use

Use when testing case-control shotgun metagenome samples (>=3 groups) for significantly changed taxa or functional features after CSS normalization, e.g. reporting ratios of changed KO/EC/reaction entities at P <= 0.01.

## When NOT to use

- Input is already a normalized feature table or compositional bias has not been removed with CSS — the article reports that without CSS, results from the two colorectal cancer datasets were not consistent
- Only two sample groups and a large sample size where metagenomeSeq's known high false-positive rate is not the target trade-off, or where an assembly-based functional profile is required instead of mapping to a gene catalog

## Inputs

- Per-sample gene frequency table from mapping shotgun reads to the IGC non-redundant gene catalog
- Taxa abundance table (e.g. from MetaPhlAn2 within CAMAMED)
- KEGG functional annotations of genes at KO, EC number and reaction levels
- Sample group labels (healthy, adenoma, carcinoma)

## Outputs

- Differential abundance results per feature at taxon, gene, KO, EC number and reaction levels with BH-adjusted P values
- Ratios of significantly changed entities per level (species 2.67%, genes 1.92%, KO groups 2.98%, EC numbers 2.55%, reactions 2.45%)

## How to apply

Map cohort shotgun reads (e.g. cohort1 colorectal adenoma/carcinoma samples) to the IGC non-redundant gene catalog (~3,354,281 mapped genes) to obtain per-sample gene frequencies, then apply cumulative sum-scaling (CSS) independently on the taxa and gene abundance tables to remove compositional bias. Map genes to KEGG to extract functional annotations at KO, EC number and reaction levels. Run the Kruskal-Wallis H test across the three disease groups with Benjamini-Hochberg correction at P <= 0.01 at each of the five annotation levels, then compute the fraction of significantly changed entities per level. The rationale is that metagenomeSeq within CAMAMED handles zero-inflation and compositional bias in sparse data, so unnormalized counts must not be fed directly to the test.

## Related tools

- **CAMAMED** (Pipeline that performs the differential abundance testing at taxon, gene, KO, EC number and reaction levels after CSS normalization) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Zero-inflated log-normal model engine used by CAMAMED for differential abundance on CSS-normalized data)
- **Docker** (Ready-to-use Docker images (camamed_pipeline, camamed_pipeline_db) so no dependencies need to be installed separately)

## Examples

```
docker run camamed/camamed_pipeline && python camamed_init
```

## Evaluation signals

- Reproduced significant-entity ratios at P <= 0.01 match Table 2 values: species 2.67%, genes 1.92%, KO groups 2.98%, EC numbers 2.55%, reactions 2.45%
- Mapped gene count equals ~3,354,281 genes when mapping cohort1 reads to the IGC catalog
- Output CSV contains differential abundance results at all five levels with BH-adjusted P values
- Functional levels (KO 2.98%, EC 2.55%, reactions 2.45%) show higher changed ratios than the gene level (1.92%), consistent with the paper's recommendation
- Gene frequencies and taxa abundances are CSS-normalized independently before testing (compositional bias removal verified per level)

## Limitations

- metagenomeSeq has a high false-positive rate for differential abundance analysis when the sample size is small
- Without applying CSS, results from the two colorectal cancer datasets were not consistent
- No software version stated for CAMAMED or its component tools; the two colorectal cancer datasets are not identified by accession

## Evidence
