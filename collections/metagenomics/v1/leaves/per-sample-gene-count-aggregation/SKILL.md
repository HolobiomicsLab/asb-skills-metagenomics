---
name: per-sample-gene-count-aggregation
description: Use when shotgun metagenomic FASTQ reads must be converted into a sample-by-gene
  count/frequency matrix via alignment against a non-redundant gene catalog, prior
  to CSS normalization or KEGG-based functional annotation.
when_to_use_negative:
- The goal is taxonomic profiling of phylogenetic diversity — CAMAMED performs that
  with MetaPhlAn v2.0 rather than gene-catalog mapping.
- No appropriate non-redundant gene catalog exists for the habitat — assembly to contigs
  followed by gene content prediction may be required instead.
- Input is already a gene count/frequency table — proceed directly to CSS normalization
  rather than re-mapping reads.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: pipeline orchestrating the mapping of metagenome sequences to non-redundant
    gene catalogs and computation of per-sample gene frequencies
  repo: https://github.com/mhnb/camamed
- name: MOSAIK
  role: hash-based aligner (v2.2.3) that maps metagenome sequences against the gene
    catalog
- name: Docker
  role: ready-to-use images camamed_pipeline and camamed_pipeline_db remove software
    installation dependencies
  repo: https://github.com/mhnb/camamed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:a27312d379a873d723d5b99be6d6b1297ca7148715a835b673ff11f215665f61
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-sample-gene-count-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CAMAMED's mapping module aligns quality-controlled metagenomic reads to a CD-HIT deduplicated non-redundant gene catalog with MOSAIK v2.2.3 and aggregates read-to-gene alignments into per-sample gene frequencies, written to gene_frequency_counts.csv.

## When to use

Use when shotgun metagenomic FASTQ reads must be converted into a sample-by-gene count/frequency matrix via alignment against a non-redundant gene catalog, prior to CSS normalization or KEGG-based functional annotation.

## When NOT to use

- The goal is taxonomic profiling of phylogenetic diversity — CAMAMED performs that with MetaPhlAn v2.0 rather than gene-catalog mapping.
- No appropriate non-redundant gene catalog exists for the habitat — assembly to contigs followed by gene content prediction may be required instead.
- Input is already a gene count/frequency table — proceed directly to CSS normalization rather than re-mapping reads.

## Inputs

- quality-controlled shotgun metagenomic reads (per sample)
- CD-HIT deduplicated non-redundant gene catalog (reference sequences)

## Outputs

- gene_frequency_counts.csv — count table with one column of gene counts/frequencies per sample
- read-to-gene alignments from MOSAIK v2.2.3

## How to apply

First launch the ready-to-use CAMAMED Docker image (camamed_pipeline or camamed_pipeline_db, the latter including MetaPhlAn2 databases) so no dependencies need local installation. Taxonomic profiling is handled separately with MetaPhlAn v2.0; for the functional/gene-level module, align the quality-controlled metagenome sequences against the CD-HIT deduplicated non-redundant gene catalog using MOSAIK v2.2.3, a hash-based aligner. MOSAIK is a mapping-based alternative preferred over assembly/contig-based gene prediction when an appropriate gene catalog exists, especially for functional-level analysis. Aggregate the read-to-gene alignments into gene frequencies per sample and write the count matrix to gene_frequency_counts.csv with one column of counts per sample. These frequencies feed downstream cumulative sum-scaling (CSS) normalization and KEGG mapping at KO, EC number, and reaction levels.

## Related tools

- **CAMAMED** (pipeline orchestrating the mapping of metagenome sequences to non-redundant gene catalogs and computation of per-sample gene frequencies) — https://github.com/mhnb/camamed
- **MOSAIK** (hash-based aligner (v2.2.3) that maps metagenome sequences against the gene catalog)
- **Docker** (ready-to-use images camamed_pipeline and camamed_pipeline_db remove software installation dependencies) — https://github.com/mhnb/camamed

## Examples

```
python camamed_init  # then follow CAMAMED-manual.pdf; mapping module aligns reads with MOSAIK v2.2.3 and writes gene_frequency_counts.csv
```

## Evaluation signals

- gene_frequency_counts.csv exists and is a readable table with one column of counts per sample
- reported gene frequencies correspond to mapped read counts over the gene catalog
- the gene catalog used is the non-redundant (CD-HIT deduplicated) version, not the raw catalog
- alignment was performed with MOSAIK v2.2.3, matching the hash-based mapping step of the CAMAMED workflow
- downstream CSS normalization at the gene level runs without error on the count matrix

## Limitations

- No software version is stated for CAMAMED or its component tools in the article or README, so reproducibility depends on the Docker image tag used
- The mapping-based approach requires an appropriate pre-built gene catalog; the article prefers mapping only 'when appropriate gene catalogs are available'
- Without subsequent CSS normalization, results may be inconsistent across datasets due to compositional bias (observed with the two colorectal cancer gut metagenome datasets)

## Evidence
