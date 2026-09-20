---
name: functional-frequency-aggregation-across-samples
description: Use when you have a per-sample gene frequency table and KO assignments
  for the gene catalog, and need per-sample counts aggregated at KO, EC number, and
  reaction levels for downstream compositional analysis.
when_to_use_negative:
- The input is already a KO/EC/reaction-level frequency table — re-aggregation would
  double-count gene-level counts.
- You need taxonomic (taxon-level) profiling, which CAMAMED handles separately via
  MetaPhlAn2 rather than through the KEGG annotation stage.
- No KEGG annotation exists for the gene catalog (e.g. genes from organisms outside
  KEGG coverage) — KO assignment would fail or be incomplete.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Hosts the annotation module that extracts KOs per gene, resolves KO→EC→reaction
    chains, and aggregates per-sample frequencies across all functional levels
  repo: https://github.com/mhnb/camamed
- name: KEGG
  role: Reference database queried to resolve KO, EC number, and reaction annotations
    for each assigned KO
- name: Docker
  role: Provides ready-to-use Docker images (camamed_pipeline / camamed_pipeline_db)
    so the KEGG annotation stage runs without separate dependency installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:38084123e73608857f5e36028ab24cf0e57fb71f33ee08033cf54982c7f0205a
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# functional-frequency-aggregation-across-samples

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In CAMAMED, after KAAS/GhostKOALA assigns KEGG orthology (KO) groups to genes from the mapped gene catalog, per-sample gene frequencies are aggregated up the KO → EC number → reaction relation chains, yielding per-sample functional frequency tables at all annotation levels.

## When to use

Use when you have a per-sample gene frequency table and KO assignments for the gene catalog, and need per-sample counts aggregated at KO, EC number, and reaction levels for downstream compositional analysis.

## When NOT to use

- The input is already a KO/EC/reaction-level frequency table — re-aggregation would double-count gene-level counts.
- You need taxonomic (taxon-level) profiling, which CAMAMED handles separately via MetaPhlAn2 rather than through the KEGG annotation stage.
- No KEGG annotation exists for the gene catalog (e.g. genes from organisms outside KEGG coverage) — KO assignment would fail or be incomplete.

## Inputs

- Per-sample gene frequency table from the upstream read-mapping module
- Gene catalog mapped to KEGG via KAAS Ver. 2.1 or GhostKOALA Ver. 2.2 (KO assignments per gene)
- KEGG database mappings for KO → EC number → reaction relation chains

## Outputs

- Per-sample functional frequency table in TSV format with one row per functional identifier (gene, KO, EC number, reaction) and one column per sample
- ko_ec_reaction_frequencies.tsv containing field columns for KO, EC number and reaction levels

## How to apply

Load the per-sample gene frequency table produced by CAMAMED's upstream read-mapping module. Submit the mapped gene catalog to KAAS Ver. 2.1 (or GhostKOALA Ver. 2.2) to assign a KO group to each gene, then query the KEGG database to resolve the KO → EC number → reaction relation chains for every assigned KO. Aggregate per-sample counts from the gene level up to KO, EC number and reaction levels, so each sample receives frequencies at all five annotation levels CAMAMED annotates (taxon, gene, KO, EC number, reaction). Write the result as a TSV with one row per functional identifier (gene, KO, EC, reaction) and one column per sample. This aggregation feeds CAMAMED's metagenomeSeq-based differential analysis with CSS normalization, which is applied at taxa and gene as well as KO, EC number and reaction levels.

## Related tools

- **CAMAMED** (Hosts the annotation module that extracts KOs per gene, resolves KO→EC→reaction chains, and aggregates per-sample frequencies across all functional levels) — https://github.com/mhnb/camamed
- **KEGG** (Reference database queried to resolve KO, EC number, and reaction annotations for each assigned KO)
- **Docker** (Provides ready-to-use Docker images (camamed_pipeline / camamed_pipeline_db) so the KEGG annotation stage runs without separate dependency installation)

## Examples

```
python camamed_init
```

## Evaluation signals

- ko_ec_reaction_frequencies.tsv exists and parses as valid TSV.
- The TSV contains field columns for KO, EC number and reaction levels.
- Per-sample gene-level row counts are consistent with the input gene frequency table.
- Every sample column carries frequencies at all functional levels (gene, KO, EC number, reaction).
- Aggregate counts at each level do not exceed the total gene-level counts for that sample (no double-counting up the relation chains).

## Limitations

- KO assignment quality depends on KAAS Ver. 2.1 or GhostKOALA Ver. 2.2; genes without KO assignments cannot propagate to EC/reaction levels.
- CAMAMED's aggregation stage requires a prior non-redundant gene catalog; without one, mapping-based functional analysis is not the appropriate approach.
- Counts at higher functional levels are compositional; downstream interpretation requires CSS normalization (without CSS, results from the two colorectal cancer datasets were not consistent).

## Evidence
