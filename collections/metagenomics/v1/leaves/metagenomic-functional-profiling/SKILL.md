---
name: metagenomic-functional-profiling
description: Use when metagenomic reads have already been mapped to a non-redundant
  gene catalog yielding per-sample gene frequencies, and the goal is to extend these
  into KO, EC number and reaction annotations with per-sample functional frequencies.
when_to_use_negative:
- Input is already a functional abundance/feature table at KO, EC or reaction level
  — no new annotation step is needed
- The analysis goal is taxonomic profiling only (taxon-level profiling is handled
  by a separate CAMAMED module, e.g. MetaPhlAn2)
- No suitable gene catalog is available — the article notes mapping-based methods
  are preferred over assembly-based approaches only 'when appropriate gene catalogs
  are available'
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline that orchestrates gene → KO → EC → reaction annotation and aggregation
    of per-sample frequencies; run via python camamed_init or Docker images camamed_pipeline
    / camamed_pipeline_db
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Deployment vehicle so no dependencies need to be installed separately (camamed_pipeline
    and camamed_pipeline_db images)
  repo: https://github.com/mhnb/camamed
- name: KEGG
  role: Reference database mapped against to extract KO, EC number and reaction annotations
    for each gene
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:ababe802d42540b0438c9cccfc70d02c8c973d27815af042cea8eb5758ce880a
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
  - KEGG
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metagenome-functional-profiling
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# metagenomic-functional-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign KEGG functional annotations (KO groups, EC numbers, reactions) to genes from a mapped gene catalog and produce per-sample frequency tables at gene, KO, EC and reaction levels, as implemented in the CAMAMED pipeline using KAAS/GhostKOALA and the KEGG database.

## When to use

Use when metagenomic reads have already been mapped to a non-redundant gene catalog yielding per-sample gene frequencies, and the goal is to extend these into KO, EC number and reaction annotations with per-sample functional frequencies.

## When NOT to use

- Input is already a functional abundance/feature table at KO, EC or reaction level — no new annotation step is needed
- The analysis goal is taxonomic profiling only (taxon-level profiling is handled by a separate CAMAMED module, e.g. MetaPhlAn2)
- No suitable gene catalog is available — the article notes mapping-based methods are preferred over assembly-based approaches only 'when appropriate gene catalogs are available'

## Inputs

- Per-sample gene frequency table (TSV) from the upstream read-mapping module
- Non-redundant gene catalog (mapped gene sequences) for KO assignment
- KEGG database (KO, EC, reaction relation data)

## Outputs

- ko_ec_reaction_frequencies.tsv — valid TSV with one row per functional identifier (gene, KO, EC number, reaction) and one column per sample
- Per-sample frequency tables at gene, KO, EC number and reaction levels

## How to apply

Load the per-sample gene frequency table produced by the upstream read-mapping module, then submit the mapped gene catalog to KAAS Ver. 2.1 (or GhostKOALA Ver. 2.2) to assign a KEGG orthology (KO) group to each gene. Query the KEGG database to resolve the KO → EC number → reaction relation chains for every assigned KO. Aggregate per-sample counts from the gene level up to KO, EC number and reaction levels so each sample receives frequencies at all annotation levels handled here (gene, KO, EC, reaction; taxon handled elsewhere in CAMAMED). Write a TSV (ko_ec_reaction_frequencies.tsv) with one row per functional identifier and one column per sample. Downstream, CAMAMED uses metagenomeSeq with cumulative sum-scaling (CSS) normalization at these levels because of its capability for handling compositional bias in sparse metagenomic data.

## Related tools

- **CAMAMED** (Pipeline that orchestrates gene → KO → EC → reaction annotation and aggregation of per-sample frequencies; run via python camamed_init or Docker images camamed_pipeline / camamed_pipeline_db) — https://github.com/mhnb/camamed
- **Docker** (Deployment vehicle so no dependencies need to be installed separately (camamed_pipeline and camamed_pipeline_db images)) — https://github.com/mhnb/camamed
- **KEGG** (Reference database mapped against to extract KO, EC number and reaction annotations for each gene)

## Examples

```
python camamed_init
```

## Evaluation signals

- ko_ec_reaction_frequencies.tsv exists and parses as valid TSV
- The output contains field columns for the KO, EC number and reaction annotation levels
- Per-sample gene-level row counts are consistent with the input gene frequency table
- Each assigned KO resolves to at least the expected EC/reaction chains in KEGG (spot-check a few KO → EC → reaction triplets)
- Frequencies aggregate correctly upward: gene-level counts sum to the KO/EC/reaction level rows per sample

## Limitations

- Mapping-based functional analysis requires an appropriate pre-existing non-redundant gene catalog; it is not an assembly-based approach
- The article/README state no software version for CAMAMED itself (only KAAS Ver. 2.1 / GhostKOALA Ver. 2.2 and MetaPhlAn2 databases in Docker images)
- Metagenomic count data are compositional and sparse; without CSS normalization, results across datasets can be inconsistent, and metagenomeSeq has a reported high false-positive rate when sample size is small
- No accessioned datasets are identified in the article for direct benchmarking of this stage

## Evidence
