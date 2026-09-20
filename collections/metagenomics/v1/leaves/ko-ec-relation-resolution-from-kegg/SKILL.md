---
name: ko-ec-relation-resolution-from-kegg
description: Use when a metagenomic gene frequency table needs functional annotation
  expanded from gene level to KEGG ortholog groups, EC numbers and reactions, as in
  the CAMAMED KEGG annotation stage.
when_to_use_negative:
- Input is a taxonomic profile or reads not yet mapped to a non-redundant gene catalog
  — the KEGG resolution stage requires per-gene frequencies as input
- Analysis only needs taxonomic profiling; this skill covers only the functional (gene,
  KO, EC, reaction) annotation levels
- A gene-level KO/EC/reaction frequency table already exists — re-running KAAS/GhostKOALA
  would only duplicate annotation
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: 'Pipeline orchestrating the KEGG annotation stage: KO assignment via KAAS/GhostKOALA,
    KO→EC→reaction resolution, and per-sample frequency aggregation to TSV'
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Delivery mechanism — the camamed_pipeline Docker images bundle dependencies
    so no separate installs (e.g. of the KEGG annotation tools) are needed
  repo: https://hub.docker.com/r/camamed/camamed_pipeline
- name: KEGG
  role: Reference database queried for KO definitions and the KO → EC number → reaction
    relation chains
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:1edfe3b19c5fcf012ddde68707bbb6f2ac2b9f98d1f523067d5ca5993f67ed68
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ko-ec-relation-resolution-from-kegg

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns KEGG orthology (KO) groups to genes from a mapped gene catalog via KAAS or GhostKOALA, then resolves KO → EC number → reaction chains in the KEGG database so per-sample frequencies can be aggregated at gene, KO, EC and reaction levels.

## When to use

Use when a metagenomic gene frequency table needs functional annotation expanded from gene level to KEGG ortholog groups, EC numbers and reactions, as in the CAMAMED KEGG annotation stage.

## When NOT to use

- Input is a taxonomic profile or reads not yet mapped to a non-redundant gene catalog — the KEGG resolution stage requires per-gene frequencies as input
- Analysis only needs taxonomic profiling; this skill covers only the functional (gene, KO, EC, reaction) annotation levels
- A gene-level KO/EC/reaction frequency table already exists — re-running KAAS/GhostKOALA would only duplicate annotation

## Inputs

- Per-sample gene frequency table (gene catalog counts from the upstream read-mapping module)
- Gene catalog sequences submitted to KAAS Ver. 2.1 or GhostKOALA Ver. 2.2 for KO assignment
- KEGG database (KO → EC → reaction relation chains)

## Outputs

- ko_ec_reaction_frequencies.tsv: per-sample functional frequency table with one row per functional identifier (gene, KO, EC number, reaction) and one column per sample

## How to apply

Load the per-sample gene frequency table produced by the upstream read-mapping module of CAMAMED (metagenome sequences already mapped to a non-redundant gene catalog). Submit the mapped gene catalog to KAAS Ver. 2.1 or GhostKOALA Ver. 2.2 to assign a KO group to each gene. Query the KEGG database to resolve, for every assigned KO, the enzyme commission (EC) numbers and reaction IDs it is associated with. Aggregate the per-sample gene-level counts up to KO, EC number and reaction levels so each sample carries frequencies at all functional annotation levels. Write the per-sample functional frequency table as TSV (ko_ec_reaction_frequencies.tsv) with one row per functional identifier and one column per sample; the compositional analysis (CSS + metagenomeSeq) is applied downstream, not here.

## Related tools

- **CAMAMED** (Pipeline orchestrating the KEGG annotation stage: KO assignment via KAAS/GhostKOALA, KO→EC→reaction resolution, and per-sample frequency aggregation to TSV) — https://github.com/mhnb/camamed
- **Docker** (Delivery mechanism — the camamed_pipeline Docker images bundle dependencies so no separate installs (e.g. of the KEGG annotation tools) are needed) — https://hub.docker.com/r/camamed/camamed_pipeline
- **KEGG** (Reference database queried for KO definitions and the KO → EC number → reaction relation chains)

## Examples

```
python camamed_init  # initialises the CAMAMED pipeline (incl. KEGG annotation stage) before running KO/EC/reaction frequency extraction
```

## Evaluation signals

- ko_ec_reaction_frequencies.tsv exists and parses as valid TSV with field columns for the KO, EC number and reaction levels
- Per-sample gene-level row counts in the output are consistent with the input gene frequency table
- Every assigned KO resolves to at least the expected EC/reaction fields drawn from the KEGG relation chain
- Each sample carries frequency values at all functional levels (gene, KO, EC, reaction) aggregated upward from gene counts
- The five annotation levels (taxon, gene, KO, EC number, reaction) are complete, with taxon handled by the separate taxonomic module

## Limitations

- No software version stated for CAMAMED or its component tools beyond KAAS Ver. 2.1 / GhostKOALA Ver. 2.2 named in the workflow
- The skill depends on KEGG database coverage; KOs without mapped EC numbers or reactions yield no downstream entries at those levels
- Per the article, correction for zero-inflation and compositional bias (CSS) is applied after this stage — raw KO/EC/reaction frequencies are compositional and should not be compared directly without normalization

## Evidence
