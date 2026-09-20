---
name: kegg-ortholog-pathway-aggregation
description: Use when you have a corrected KO abundance table from shotgun metagenomes
  and need higher-level KEGG pathway/module functional profiles rather than ortholog-level
  abundances.
when_to_use_negative:
- Input is raw gene or read counts that have not yet been mapped to KOs and corrected
  with MUSiCC — run the ortholog counting and correction steps first.
- You need taxonomic profiles or species-level functional stratification — EMPANADA
  aggregation is purely KO-to-KEGG-category summarization.
- Your target categories are not KEGG modules/pathways (e.g. UniRef90-based or eggNOG
  functional classes) — the mapping files used here are KEGG BRITE-derived.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: EMPANADA
  role: Distributes each KO's abundance among all KEGG modules/pathways it belongs
    to, weighted by support for the presence of each category
- name: MetaLAFFA
  role: Pipeline implementing the ortholog_aggregation step and final output writing
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Snakemake
  role: Orchestrates the ortholog_aggregation step
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: prepare_databases.py
  role: Downloads KEGG ko-to-module and ko-to-pathway mappings via the -km option
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:79ba75ef8f567095d4ebee26c4d8f20a140ef364c2b28f9ef51124665b0c60dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - EMPANADA
  - MetaLAFFA
  - Python
  - Snakemake
  - prepare_databases.py
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# kegg-ortholog-pathway-aggregation

## Summary

Aggregates MUSiCC-corrected KEGG ortholog (KO) abundances into KEGG module- and pathway-level functional profiles using EMPANADA, weighted by the supporting evidence for each higher-level category.

## When to use

Use when you have a corrected KO abundance table from shotgun metagenomes and need higher-level KEGG pathway/module functional profiles rather than ortholog-level abundances.

## When NOT to use

- Input is raw gene or read counts that have not yet been mapped to KOs and corrected with MUSiCC — run the ortholog counting and correction steps first.
- You need taxonomic profiles or species-level functional stratification — EMPANADA aggregation is purely KO-to-KEGG-category summarization.
- Your target categories are not KEGG modules/pathways (e.g. UniRef90-based or eggNOG functional classes) — the mapping files used here are KEGG BRITE-derived.

## Inputs

- MUSiCC-corrected KO abundance table (inter-sample normalized, intra-sample corrected)
- KEGG ko-to-module mapping file (from prepare_databases.py -km)
- KEGG ko-to-pathway mapping file (from prepare_databases.py -km)

## Outputs

- KO-level functional profile TSV (carried through from correction step)
- KEGG module-level abundance table (TSV in output/final)
- KEGG pathway-level abundance table (TSV in output/final)

## How to apply

Load the MUSiCC-corrected KO abundance table produced by the ortholog abundance correction step (inter-sample normalization with intra-sample correction). Load the KEGG ko-to-module and ko-to-pathway mapping files downloaded via prepare_databases.py (-km), which define KO membership in higher-level functional categories from the KEGG BRITE hierarchy. Run the MetaLAFFA ortholog_aggregation step (configured in config/steps/ortholog_aggregation.py and orchestrated by the Snakemake pipeline), invoking EMPANADA to distribute each KO's abundance among all functional categories it belongs to, weighted by the support for the presence of each category. Write the aggregated module- and pathway-level tables alongside the corrected KO table to the pipeline's final_output_directory (output/final) as tab-separated files.

## Related tools

- **EMPANADA** (Distributes each KO's abundance among all KEGG modules/pathways it belongs to, weighted by support for the presence of each category)
- **MetaLAFFA** (Pipeline implementing the ortholog_aggregation step and final output writing) — https://github.com/borenstein-lab/MetaLAFFA
- **Snakemake** (Orchestrates the ortholog_aggregation step) — https://github.com/borenstein-lab/MetaLAFFA
- **prepare_databases.py** (Downloads KEGG ko-to-module and ko-to-pathway mappings via the -km option) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
prepare_databases.py -hr -km -u
```

## Evaluation signals

- final_output_directory (output/final) contains non-empty TSV tables for KO, module, and pathway abundances
- Each output table passes file_exists and file_format_is TSV checks with non-zero row counts
- Module and pathway tables contain valid KEGG functional identifiers (module/pathway IDs from the KEGG BRITE mappings)
- Aggregated pathway/module values are consistent with the corrected KO table under EMPANADA's support-weighted distribution (no KO abundance lost or duplicated beyond category overlap weighting)
- Pipeline summary statistics for the aggregation step are generated alongside the final tables

## Limitations

- Aggregation quality depends on the KEGG ko-to-module and ko-to-pathway mappings downloaded with -km; mappings must be prepared in advance (database preparation can take several hours, ~40GB RAM, and ~150–300GB disk space).
- KOs belonging to multiple categories have their abundance distributed across categories, so pathway/module abundances are support-weighted estimates rather than simple sums.
- MUSiCC may fail with an 'undefined symbol: omp_get_num_procs' error in some environments (documented FAQ issue), which blocks the upstream corrected-KO input.

## Evidence

- [readme] Aggregating orthology groups into higher-level functional descriptions: "9. Aggregating orthology groups into higher-level functional descriptions"
- [readme] aggregation step description: "Aggregating orthologs into higher-level groupings (e.g. pathways)"
- [readme] km mapping download: "-km`: Download KEGG ko-to-module and ko-to-pathway mappings."
- [readme] final outputs: "Final outputs location and descriptions"
- [abstract] functional profiles output: "MetaLAFFA, a functional annotation pipeline designed to take unfiltered shotgun metagenomic data as input and generate functional profiles"
