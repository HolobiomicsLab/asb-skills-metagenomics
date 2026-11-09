---
name: docker-based-pipeline-execution
description: Use when a user wants to execute the full CAMAMED workflow (quality-controlled
  reads → MetaPhlAn2/MOSAIK → KEGG KO/EC/reaction annotation) without installing software
  dependencies, or when reproducible containerized execution is required.
when_to_use_negative:
- When the user only needs one component tool (e.g., standalone MetaPhlAn2 or MOSAIK)
  with custom parameters — the container fixes tool versions and wraps them in the
  CAMAMED workflow
- When downstream differential-abundance analysis is already done and the user only
  needs CSS-normalized statistics rather than pipeline execution
- When no non-redundant gene catalog is available, since the mapping-based MOSAIK
  stage requires one
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: The pipeline executed inside the Docker image; orchestrates taxonomic and
    functional profiling stages
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Container runtime that ships camamed_pipeline / camamed_pipeline_db images
    so no dependencies need separate installation
- name: MetaPhlAn
  role: Taxonomic profiling stage run inside the container (v2.0); databases preloaded
    only in the _db image
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:0db3c938fe87027845bc8805df4f9487837cd71d6e56143b597efa5f8bc5a90a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# docker-based-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the CAMAMED metagenomic analysis pipeline (taxonomic profiling with MetaPhlAn2, MOSAIK mapping to a gene catalog, and KEGG-based functional annotation) inside a prebuilt Docker image so that no component dependencies (MetaPhlAn, MOSAIK, KAAS/GhostKOALA tooling) need to be installed locally.

## When to use

Use when a user wants to execute the full CAMAMED workflow (quality-controlled reads → MetaPhlAn2/MOSAIK → KEGG KO/EC/reaction annotation) without installing software dependencies, or when reproducible containerized execution is required.

## When NOT to use

- When the user only needs one component tool (e.g., standalone MetaPhlAn2 or MOSAIK) with custom parameters — the container fixes tool versions and wraps them in the CAMAMED workflow
- When downstream differential-abundance analysis is already done and the user only needs CSS-normalized statistics rather than pipeline execution
- When no non-redundant gene catalog is available, since the mapping-based MOSAIK stage requires one

## Inputs

- Quality-controlled metagenomic reads (per sample)
- Non-redundant gene catalog (e.g., CD-HIT deduplicated IGC-style catalog) for MOSAIK mapping
- Choice of Docker image: camamed_pipeline or camamed_pipeline_db (with MetaPhlAn2 databases)

## Outputs

- Per-sample taxonomic profiles (MetaPhlAn v2.0)
- Per-sample gene frequency table (gene_frequency_counts.csv)
- Per-sample functional frequency table (ko_ec_reaction_frequencies.tsv) at gene, KO, EC number and reaction levels

## How to apply

Pull one of the two published Docker images from Docker Hub — camamed_pipeline (without MetaPhlAn2 databases) or camamed_pipeline_db (with MetaPhlAn2 databases) — choosing the latter when taxonomic profiling will be run so the database ships with the image. Initialize the environment by running `python camamed_init` as the first step before any pipeline stage, then consult CAMAMED-manual.pdf for stage-specific arguments. Execute the pipeline stages in order: MetaPhlAn v2.0 taxonomic profiling on quality-controlled reads, MOSAIK v2.2.3 alignment of metagenomic reads against the (CD-HIT deduplicated) non-redundant gene catalog to obtain per-sample gene frequencies, and KAAS Ver. 2.1 / GhostKOALA Ver. 2.2 KO assignment followed by KEGG KO→EC→reaction resolution and aggregation. The rationale is that the container bundles all tools so the nonprofessional user is 'liberated from being engaged in potential software installation obstacles'; versions of individual tools are fixed inside the image since no standalone version is stated for CAMAMED itself.

## Related tools

- **CAMAMED** (The pipeline executed inside the Docker image; orchestrates taxonomic and functional profiling stages) — https://github.com/mhnb/camamed
- **Docker** (Container runtime that ships camamed_pipeline / camamed_pipeline_db images so no dependencies need separate installation)
- **MetaPhlAn** (Taxonomic profiling stage run inside the container (v2.0); databases preloaded only in the _db image)

## Examples

```
docker pull camamed/camamed_pipeline_db && python camamed_init
```

## Evaluation signals

- The `python camamed_init` step completed before any pipeline stage was executed
- gene_frequency_counts.csv exists, is readable, has one column of counts per sample, and frequencies correspond to mapped read counts over the gene catalog
- ko_ec_reaction_frequencies.tsv exists, is valid TSV, contains field columns for KO, EC number and reaction levels, and gene-level row counts match the input gene frequency table
- The chosen image matches the need: camamed_pipeline_db when MetaPhlAn2 databases were not supplied externally; camamed_pipeline when they were
- No separate tool installations were needed — all stages ran solely from the container image

## Limitations

- No software version is stated for CAMAMED or its component tools in the article; tool versions are implicit in the Docker image
- No changelog exists for the repository, so tracking image-to-image changes is difficult
- The base camamed_pipeline image lacks MetaPhlAn2 databases; using it for taxonomic profiling requires supplying databases separately

## Evidence

- [readme] To run CAMAMED, first run the following command
     python camamed_init: "To run CAMAMED, first run the following command
     python camamed_init"
