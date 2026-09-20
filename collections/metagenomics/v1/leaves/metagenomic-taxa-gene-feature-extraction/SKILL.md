---
name: metagenomic-taxa-gene-feature-extraction
description: Use when shotgun metagenome reads must be converted into taxon- and gene-level
  abundance tables and differentially abundant colorectal-cancer-style biomarkers
  must be identified while correcting compositional bias.
when_to_use_negative:
- When no appropriate non-redundant gene catalog exists for the samples — mapping-based
  functional analysis is then not preferred and assembly-based gene prediction should
  be considered instead
- 'When samples need annotation at levels CAMAMED does not output — the pipeline annotates
  at five levels only: taxon, gene, KO, EC number and reaction'
- When comparing uncorrected vs CSS-corrected results already exists — the skill covers
  feature extraction and CSS-corrected marker discovery, not re-evaluating existing
  consistency metrics
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: End-to-end pipeline that maps reads to gene catalogs, produces taxon/gene
    abundance tables, applies CSS, and runs metagenomeSeq marker discovery
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Zero-inflated log-normal differential abundance model applied to CSS-normalized
    taxa/gene counts to extract case-control markers
- name: Docker
  role: Container runtime for CAMAMED; two images (camamed_pipeline without MetaPhlAn2
    databases, camamed_pipeline_db with them) avoid manual dependency installation
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:12b8f00422ee5e26f3e7ca486ff3c761e43d1ed04424b3e131b73672ddf9a626
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metagenomic-taxa-gene-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract taxon- and gene-level abundance count tables from shotgun metagenome samples and identify case-control biomarker features using CSS compositional correction with the metagenomeSeq zero-inflated log-normal model, as implemented in the CAMAMED pipeline.

## When to use

Use when shotgun metagenome reads must be converted into taxon- and gene-level abundance tables and differentially abundant colorectal-cancer-style biomarkers must be identified while correcting compositional bias.

## When NOT to use

- When no appropriate non-redundant gene catalog exists for the samples — mapping-based functional analysis is then not preferred and assembly-based gene prediction should be considered instead
- When samples need annotation at levels CAMAMED does not output — the pipeline annotates at five levels only: taxon, gene, KO, EC number and reaction
- When comparing uncorrected vs CSS-corrected results already exists — the skill covers feature extraction and CSS-corrected marker discovery, not re-evaluating existing consistency metrics

## Inputs

- Shotgun metagenome sequencing reads (FASTQ) from case-control samples
- Non-redundant gene catalog for mapping-based gene-frequency estimation
- KEGG database for gene functional annotation (KO, EC, reaction)
- Sample metadata with case/control labels

## Outputs

- Taxon-level abundance count table
- Gene-level abundance count table
- CSS-normalized taxa and gene tables fitted with the metagenomeSeq zero-inflated log-normal model
- Differential abundance marker taxa and gene lists (potential biomarkers)
- Cross-dataset marker consistency summary (e.g. css_cross_dataset_validation.csv)

## How to apply

Map metagenome sequences to non-redundant gene catalogs to obtain gene frequencies, then aggregate to taxon and gene abundance count tables. Apply cumulative sum-scaling (CSS) normalization independently at the taxa and gene levels, because metagenomic count data are compositional and need systematic variability removed before statistical testing. Fit the metagenomeSeq zero-inflated log-normal differential abundance model to the CSS-corrected tables to extract differentially abundant marker taxa and genes between case and control samples; CAMAMED additionally propagates gene annotations to KO, EC number and reaction levels via the KEGG database. Prefer mapping-based analysis over assembly when an appropriate gene catalog is available, especially for functional-level analysis. To validate marker reproducibility, run the identical CSS-corrected analysis on a second independent dataset and compare marker-set overlap against an uncorrected (no-CSS) run; the CAMAMED paper expects corrected markers to agree across datasets while uncorrected ones do not.

## Related tools

- **CAMAMED** (End-to-end pipeline that maps reads to gene catalogs, produces taxon/gene abundance tables, applies CSS, and runs metagenomeSeq marker discovery) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Zero-inflated log-normal differential abundance model applied to CSS-normalized taxa/gene counts to extract case-control markers)
- **Docker** (Container runtime for CAMAMED; two images (camamed_pipeline without MetaPhlAn2 databases, camamed_pipeline_db with them) avoid manual dependency installation) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
python camamed_init
```

## Evaluation signals

- CSS-corrected marker sets from two independent colorectal cancer gut metagenome datasets show substantially greater cross-dataset agreement than uncorrected marker sets
- The uncorrected (no-CSS) comparison exhibits inconsistency between datasets, matching the paper's reported failure mode
- Output summary file (e.g. css_cross_dataset_validation.csv) exists and contains consistency metrics for both CSS-corrected and uncorrected comparisons
- Abundance tables are produced at all five CAMAMED annotation levels (taxon, gene, KO, EC number, reaction) when functional annotation is requested
- Gene counts derive from mapping to a non-redundant gene catalog, with gene frequencies successfully obtained for every sample

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance analysis when sample size is small
- The two colorectal cancer gut metagenome datasets are not identified by accession, so input data provenance must be supplied externally
- No software version is stated for CAMAMED or its component tools, complicating exact reproducibility
- Correction for zero-inflation and compositional bias is largely neglected in other pipelines, so results will differ from pipelines that omit CSS

## Evidence
