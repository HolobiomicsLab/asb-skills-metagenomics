---
name: metagenomic-read-alignment-to-gene-catalog
description: Use when you have QC'd shotgun metagenomic FASTQ reads and a non-redundant
  gene catalog (e.g., the IGC catalog), and you need per-sample gene frequencies as
  input to compositional differential-abundance analysis.
when_to_use_negative:
- Your aim is taxonomic profiling of phylogenetic diversity — that is done separately
  with MetaPhlAn v2.0, not by catalog mapping.
- No appropriate non-redundant gene catalog is available for the environment under
  study; the article prefers mapping-based analysis only 'when appropriate gene catalogs
  are available'.
- You need contig-level reconstruction or novel gene discovery — this mapping approach
  does not assemble reads or predict new gene content.
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CAMAMED
  role: Pipeline that performs the catalog mapping and computes per-sample gene frequencies
  repo: https://github.com/mhnb/camamed
- name: MOSAIK
  role: Hash-based aligner (v2.2.3) used to map metagenome sequences to the non-redundant
    gene catalog
- name: Docker
  role: Deployment of CAMAMED via the camamed_pipeline / camamed_pipeline_db images,
    avoiding dependency installation
- name: metagenomeSeq
  role: 'Downstream consumer: CSS-normalized gene frequency counts are analyzed at
    taxa, gene, KO, EC number and reaction levels'
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:63591353cc032b58eefd4d7d9584605bc4aad721a66484cac7a5a7c9e3d2a20a
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
  - MOSAIK
  - MetaPhlAn
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metagenomic-read-alignment-to-gene-catalog

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map quality-controlled shotgun metagenomic reads to a non-redundant (CD-HIT deduplicated) gene catalog with a hash-based aligner to compute per-sample gene frequencies, the foundation for CAMAMED's functional (gene, KO, EC, reaction) analysis.

## When to use

Use when you have QC'd shotgun metagenomic FASTQ reads and a non-redundant gene catalog (e.g., the IGC catalog), and you need per-sample gene frequencies as input to compositional differential-abundance analysis.

## When NOT to use

- Your aim is taxonomic profiling of phylogenetic diversity — that is done separately with MetaPhlAn v2.0, not by catalog mapping.
- No appropriate non-redundant gene catalog is available for the environment under study; the article prefers mapping-based analysis only 'when appropriate gene catalogs are available'.
- You need contig-level reconstruction or novel gene discovery — this mapping approach does not assemble reads or predict new gene content.

## Inputs

- Quality-controlled shotgun metagenomic reads per sample
- Non-redundant (CD-HIT deduplicated) gene catalog, e.g., the IGC catalog
- Docker image camamed_pipeline (without MetaPhlAn2 databases) or camamed_pipeline_db (with databases)

## Outputs

- Per-sample gene frequency count matrix written to gene_frequency_counts.csv (one column of counts per sample)
- Read-to-gene alignments over the gene catalog

## How to apply

Run CAMAMED via its ready-to-use Docker image (camamed_pipeline or camamed_pipeline_db) so no dependencies must be installed; first initialize with `python camamed_init`. Align the metagenome sequences against the non-redundant gene catalog using MOSAIK v2.2.3, a hash-based aligner (the paper reports ~3,354,281 mapped genes for the colorectal cancer cohort1 dataset). Aggregate read-to-gene alignments into per-sample gene frequencies and write the count matrix to gene_frequency_counts.csv. Rationale: mapping-based analysis against a catalog is preferred over assembly-based contig/gene-prediction approaches when an appropriate gene catalog exists, especially for functional-level analysis.

## Related tools

- **CAMAMED** (Pipeline that performs the catalog mapping and computes per-sample gene frequencies) — https://github.com/mhnb/camamed
- **MOSAIK** (Hash-based aligner (v2.2.3) used to map metagenome sequences to the non-redundant gene catalog)
- **Docker** (Deployment of CAMAMED via the camamed_pipeline / camamed_pipeline_db images, avoiding dependency installation)
- **metagenomeSeq** (Downstream consumer: CSS-normalized gene frequency counts are analyzed at taxa, gene, KO, EC number and reaction levels)

## Examples

```
python camamed_init  # then run CAMAMED via the camamed_pipeline Docker image to map reads with MOSAIK and write gene_frequency_counts.csv
```

## Evaluation signals

- gene_frequency_counts.csv exists and is a readable table with one column of counts per sample
- Reported frequencies correspond to mapped read counts over the gene catalog
- Expected mapped-gene scale: ~3,354,281 genes mapped for the cohort1 colorectal cancer dataset
- Gene frequencies feed CSS normalization without errors, confirming a complete count matrix
- Docker image variant chosen matches whether MetaPhlAn2 databases are bundled (camamed_pipeline vs camamed_pipeline_db)

## Limitations

- No software version stated for CAMAMED or its component tools in the article; MOSAIK v2.2.3 and MetaPhlAn v2.0 are named only in derived task documentation
- Mapping-based analysis requires an appropriate pre-existing gene catalog; it cannot discover genes absent from the catalog
- The two colorectal cancer gut metagenome datasets used are not identified by accession, limiting direct reuse of the benchmark
- Downstream differential-abundance results are inconsistent if compositional bias is not removed (CSS) after mapping

## Evidence
