---
name: taxonomic-profiling-with-marker-genes
description: Use when metagenomic sequencing reads (post quality-control) need organism-level
  phylogenetic characterization prior to or alongside gene-catalog mapping, as in
  the CAMAMED case-control colorectal cancer gut metagenome workflow.
when_to_use_negative:
- Input is already a taxonomic or gene abundance feature table — the profiling step
  expects raw/quality-controlled reads.
- No appropriate gene catalog or marker-gene database is available; the article states
  mapping-based methods are preferred only when appropriate gene catalogs exist.
- The goal is functional annotation at KO/EC/reaction level without any taxonomic
  component — that uses the KEGG mapping stage instead.
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Composition-aware pipeline that orchestrates the taxonomic profiling step
    and subsequent CSS normalization at taxon and gene levels
  repo: https://github.com/mhnb/camamed
- name: MetaPhlAn
  role: Marker-gene taxonomic profiler (v2.0) executed on quality-controlled reads
    within the pipeline
- name: Docker
  role: Distributed as camamed_pipeline / camamed_pipeline_db images so MetaPhlAn2
    databases and dependencies need no separate installation
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:4a9487659bdeafad5fc6a6fea15acf6604566adea1988a4d1eb79e0179513e64
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
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# taxonomic-profiling-with-marker-genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs taxonomic profiling of quality-controlled metagenomic reads with MetaPhlAn v2.0 inside the CAMAMED pipeline, describing the phylogenetic diversity of microorganisms in each sample as the first analytical stage before mapping-based functional analysis.

## When to use

Use when metagenomic sequencing reads (post quality-control) need organism-level phylogenetic characterization prior to or alongside gene-catalog mapping, as in the CAMAMED case-control colorectal cancer gut metagenome workflow.

## When NOT to use

- Input is already a taxonomic or gene abundance feature table — the profiling step expects raw/quality-controlled reads.
- No appropriate gene catalog or marker-gene database is available; the article states mapping-based methods are preferred only when appropriate gene catalogs exist.
- The goal is functional annotation at KO/EC/reaction level without any taxonomic component — that uses the KEGG mapping stage instead.

## Inputs

- Quality-controlled metagenomic sequencing reads (FASTQ) per sample
- MetaPhlAn2 marker-gene database (bundled in the camamed_pipeline_db Docker image)

## Outputs

- Per-sample taxonomic profile of microbial phylogenetic diversity (taxon abundance table)
- Taxon-level abundance data subsequently normalized with cumulative sum-scaling for compositional data analysis

## How to apply

Launch the ready-to-use CAMAMED Docker image (camamed_pipeline_db includes the MetaPhlAn2 databases; camamed_pipeline omits them) so no dependencies must be installed separately. Initialize with `python camamed_init`, then run MetaPhlAn v2.0 on the quality-controlled reads to produce per-sample taxonomic profiles of the phylogenetic diversity of microorganisms. In the source card workflow this profiling stage is paired with MOSAIK v2.2.3 mapping to a CD-HIT-deduplicated gene catalog to derive per-sample gene frequencies (gene_frequency_counts.csv). Rationale: profiling with marker genes avoids assembling reads and gives composition-aware input for downstream cumulative sum-scaling (CSS) normalization at the taxon level, which CAMAMED requires for reproducible differential-abundance results.

## Related tools

- **CAMAMED** (Composition-aware pipeline that orchestrates the taxonomic profiling step and subsequent CSS normalization at taxon and gene levels) — https://github.com/mhnb/camamed
- **MetaPhlAn** (Marker-gene taxonomic profiler (v2.0) executed on quality-controlled reads within the pipeline)
- **Docker** (Distributed as camamed_pipeline / camamed_pipeline_db images so MetaPhlAn2 databases and dependencies need no separate installation) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
python camamed_init
```

## Evaluation signals

- Taxonomic profile output exists and covers the phylogenetic diversity of microorganisms for each input sample.
- The companion gene_frequency_counts.csv (if the full mapping stage was run) exists, is readable, and has one column of counts per sample.
- Profiles derive from quality-controlled reads and the correct MetaPhlAn2 database variant (camamed_pipeline_db ships the databases).
- Downstream taxa-level abundance data have had compositional bias removal performed independently using the CSS method before differential analysis.
- Docker image usage is confirmed so no software dependencies were installed separately by the user.

## Limitations

- The camamed_pipeline image lacks the MetaPhlAn2 databases, which must be supplied separately if camamed_pipeline_db is not used.
- The article/README state no software version for pipeline components beyond MetaPhlAn2 in the image naming; version provenance must be tracked manually.
- Without applying CSS to the resulting taxon-level data, results from the two colorectal cancer datasets were not consistent, per the article's discussion.

## Evidence
