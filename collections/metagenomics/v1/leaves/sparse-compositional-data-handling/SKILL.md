---
name: sparse-compositional-data-handling
description: Use when mapping-based metagenomic gene frequency tables are sparse and
  compositional, and downstream taxonomic or KO/EC/reaction-level profiling requires
  correction for compositional and zero-inflation bias.
when_to_use_negative:
- The input data are already relative frequencies or otherwise compositionally normalized
  abundance tables, e.g. counts pre-converted by a CoDA method such as RAIDA
- The count matrix is dense RNA-seq-style data better handled by RNA-seq-oriented
  methods such as DESeq2 or edgeR
- Differential abundance testing with a small sample size is the sole goal, since
  metagenomeSeq has a reported high false-positive rate in that setting
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline that implements the filtering, gene-length correction, and CSS normalization
    workflow and writes the normalized tables
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Provides cumulative sum-scaling (CSS) normalization with a zero-inflated log-normal
    model to correct compositional bias in sparse gene and taxa abundance data
- name: Docker
  role: Distributed as Docker images (camamed_pipeline and camamed_pipeline_db) so
    no separate dependency installation is needed
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:dc2f41b2b2c3332d530f9af467506b6edf6ab75de3755917ca709a41850dc69c
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sparse-compositional-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalization strategy for sparse, compositional metagenomic count data as implemented in CAMAMED: low-count gene filtering, gene-length correction, and cumulative sum-scaling (CSS) via metagenomeSeq to remove compositional bias before taxonomic and functional profiling.

## When to use

Use when mapping-based metagenomic gene frequency tables are sparse and compositional, and downstream taxonomic or KO/EC/reaction-level profiling requires correction for compositional and zero-inflation bias.

## When NOT to use

- The input data are already relative frequencies or otherwise compositionally normalized abundance tables, e.g. counts pre-converted by a CoDA method such as RAIDA
- The count matrix is dense RNA-seq-style data better handled by RNA-seq-oriented methods such as DESeq2 or edgeR
- Differential abundance testing with a small sample size is the sole goal, since metagenomeSeq has a reported high false-positive rate in that setting

## Inputs

- Gene frequency count table from mapping metagenome reads to a non-redundant gene catalog
- Gene length annotations for length normalization
- KEGG-based functional annotation of genes (KO groups, EC numbers, reactions)

## Outputs

- css_normalized_frequencies.tsv containing normalized gene, species, and functional-level (KO, EC number, reaction) frequency tables

## How to apply

Load the gene frequency count table produced by mapping metagenome sequences to a non-redundant gene catalog. Filter out genes with fewer than five mapped reads summed across all samples, then divide each gene's frequency by its gene length to obtain length-normalized frequencies. Apply cumulative sum-scaling (CSS) normalization with the metagenomeSeq package at the gene level, exploiting its zero-inflated log-normal model for sparse compositional data, and repeat CSS at the species (taxon) level. Aggregate normalized gene frequencies into KO, EC number, and reaction frequency tables by summing the normalized frequencies of genes within each functional annotation category mapped via KEGG. The rationale is that compositional bias and zero-inflation in metagenomic counts must be corrected before biomarker discovery; without CSS, results from the two colorectal cancer gut metagenome datasets were inconsistent.

## Related tools

- **CAMAMED** (Pipeline that implements the filtering, gene-length correction, and CSS normalization workflow and writes the normalized tables) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Provides cumulative sum-scaling (CSS) normalization with a zero-inflated log-normal model to correct compositional bias in sparse gene and taxa abundance data)
- **Docker** (Distributed as Docker images (camamed_pipeline and camamed_pipeline_db) so no separate dependency installation is needed) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
python camamed_init
```

## Evaluation signals

- All genes with fewer than five mapped reads summed across samples are absent from the normalized output table
- Gene frequencies are length-normalized before CSS is applied (each gene divided by its gene length)
- Both gene-level and species-level tables are CSS-normalized, and KO/EC/reaction tables are sums of normalized subset-gene frequencies
- Output file css_normalized_frequencies.tsv exists and contains gene, species, and functional-level frequency columns
- Reproducibility check: applying the pipeline with and without CSS should change results (article reports inconsistency between colorectal cancer datasets without CSS)

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance analysis when the sample size is small
- No software version is stated for CAMAMED or its component tools, complicating exact reproducibility
- Correction for zero-inflation and compositional bias is largely neglected in other pipelines, so cross-pipeline comparisons of normalized outputs are not straightforward

## Evidence

- [methods] CAMAMED's normalization module removes genes with fewer than five mapped reads across all samples, divides each gene frequency by gene length: [evidence span withheld — anchor status: not_found]
