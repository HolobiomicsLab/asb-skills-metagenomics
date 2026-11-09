---
name: kegg-orthology-annotation-of-mapped-genes
description: Use when a non-redundant gene catalog has already been read-mapped to
  produce per-sample gene frequencies, and the analysis goal is functional profiling
  beyond the gene level at KO, EC number and reaction levels.
when_to_use_negative:
- The analysis goal is only taxonomic profiling (handled by the MetaPhlAn2-based taxon
  level of CAMAMED, not KEGG annotation).
- No pre-built gene catalog or upstream gene frequency table exists — assembly-based
  functional profiling (assemble reads to contigs, then predict gene content) would
  be needed instead.
- The data is already annotated at KO, EC and reaction levels; re-running KAAS/GhostKOALA
  against KEGG would be redundant.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: CAMAMED
  role: Pipeline whose KEGG annotation stage performs KO assignment, KO → EC → reaction
    extraction, and per-sample frequency aggregation from gene to KO/EC/reaction levels
  repo: https://github.com/mhnb/camamed
- name: KAAS
  role: KEGG Automatic Annotation Server (Ver. 2.1) used to assign a KEGG orthology
    (KO) group to each gene in the catalog
- name: GhostKOALA
  role: Alternative server (Ver. 2.2) for KO assignment to the mapped genes
- name: KEGG
  role: Reference database queried to resolve KO → EC number → reaction relation chains
- name: Docker
  role: Docker images (camamed_pipeline / camamed_pipeline_db) avoid separate installation
    of pipeline dependencies
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:4b53b87aca35afb800a79d83113401b6f4141eca2a3173d98c5f185e9ecd45c0
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
  - GhostKOALA
  - KAAS
  - KEGG
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kegg-orthology-annotation-of-mapped-genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates genes from a mapping-based metagenomic gene catalog to the KEGG universe: assigns KO groups via KAAS/GhostKOALA, resolves KO → EC number → reaction chains, and aggregates per-sample gene frequencies upward so every sample receives frequencies at gene, KO, EC and reaction levels.

## When to use

Use when a non-redundant gene catalog has already been read-mapped to produce per-sample gene frequencies, and the analysis goal is functional profiling beyond the gene level at KO, EC number and reaction levels.

## When NOT to use

- The analysis goal is only taxonomic profiling (handled by the MetaPhlAn2-based taxon level of CAMAMED, not KEGG annotation).
- No pre-built gene catalog or upstream gene frequency table exists — assembly-based functional profiling (assemble reads to contigs, then predict gene content) would be needed instead.
- The data is already annotated at KO, EC and reaction levels; re-running KAAS/GhostKOALA against KEGG would be redundant.

## Inputs

- Per-sample gene frequency table from the upstream CAMAMED read-mapping module
- Non-redundant gene catalog of mapped metagenomic genes
- KEGG database (via KAAS Ver. 2.1 or GhostKOALA Ver. 2.2) for KO assignment and KO → EC → reaction resolution

## Outputs

- ko_ec_reaction_frequencies.tsv — valid TSV with field columns for KO, EC number and reaction levels, one row per functional identifier and one column per sample
- Per-sample gene-level frequency rows consistent with the input gene frequency table

## How to apply

Load the per-sample gene frequency table produced by the upstream read-mapping module, then submit the mapped gene catalog to KAAS Ver. 2.1 (or GhostKOALA Ver. 2.2) to assign a KEGG orthology (KO) group to each gene. Query the KEGG database to resolve the KO → EC number → reaction relation chains for every assigned KO. Aggregate the per-sample counts from the gene level up to KO, EC number and reaction levels, so that each sample carries frequencies at all five CAMAMED annotation levels (taxon is handled separately). Write the result as a per-sample functional frequency table in TSV format with one row per functional identifier (gene, KO, EC, reaction) and one column per sample. The rationale is that CAMAMED's mapping-based strategy extracts annotations 'at different functional levels such as KEGG ortholog groups, enzyme commission numbers and reactions', enabling downstream compositional analysis (CSS + metagenomeSeq) and biomarker discovery at any of these levels.

## Related tools

- **CAMAMED** (Pipeline whose KEGG annotation stage performs KO assignment, KO → EC → reaction extraction, and per-sample frequency aggregation from gene to KO/EC/reaction levels) — https://github.com/mhnb/camamed
- **KAAS** (KEGG Automatic Annotation Server (Ver. 2.1) used to assign a KEGG orthology (KO) group to each gene in the catalog)
- **GhostKOALA** (Alternative server (Ver. 2.2) for KO assignment to the mapped genes)
- **KEGG** (Reference database queried to resolve KO → EC number → reaction relation chains)
- **Docker** (Docker images (camamed_pipeline / camamed_pipeline_db) avoid separate installation of pipeline dependencies) — https://hub.docker.com/r/camamed/camamed_pipeline_db

## Examples

```
python camamed_init
```

## Evaluation signals

- Verify that ko_ec_reaction_frequencies.tsv exists and is a valid TSV file.
- Confirm the TSV contains field columns for the KO, EC number and reaction levels.
- Check that per-sample gene-level row counts are consistent with the input gene frequency table.
- Confirm each sample receives frequencies at all five annotation levels (taxon handled elsewhere; gene, KO, EC, reaction here).
- Spot-check that each assigned KO resolves to plausible EC number and reaction IDs in the KEGG relation chains.

## Limitations

- No software version is stated for CAMAMED itself; only KAAS Ver. 2.1 / GhostKOALA Ver. 2.2 are versioned in the workflow.
- The annotation stage depends on a pre-existing gene catalog and upstream per-sample gene frequency table; it cannot run standalone on raw reads.
- Functional annotation is restricted to the KEGG universe (KO, EC, reaction); other functional databases are not covered.
- The two colorectal cancer gut metagenome datasets used in the paper's case study are not identified by accession, limiting reproducibility checks.

## Evidence
