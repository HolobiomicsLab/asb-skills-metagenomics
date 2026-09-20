---
name: kegg-brite-hierarchy-usage
description: Use when you have a corrected KEGG orthology group (KO) abundance table
  and need higher-level KEGG module and pathway abundance profiles, or need to interpret
  KOs within the KEGG BRITE hierarchy.
when_to_use_negative:
- Input is raw FASTQ reads or uncorrected gene counts — the aggregation step expects
  the MUSiCC-corrected KO table, not upstream read-processing outputs
- Functional categories outside the KEGG BRITE hierarchy (e.g. CARD antibiotic resistance
  genes or UniRef90-based groupings) — the ko-to-module/ko-to-pathway mappings only
  cover KEGG modules and pathways
- Assembly- or contig-based functional annotation rather than read-based KO profiles
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: MetaLAFFA
  role: Pipeline providing the ortholog_aggregation step and final_output_directory
    structure for the aggregated profiles
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: EMPANADA
  role: Performs the KO-to-module/pathway aggregation, weighting each KO's abundance
    by support for the presence of each functional category
- name: Snakemake
  role: Orchestrates the ortholog_aggregation step in the MetaLAFFA pipeline
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: prepare_databases.py
  role: Downloads the KEGG ko-to-module and ko-to-pathway mappings via the -km option
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:22628252186cdadea9e6ab843b04da3b9887d20feb4380898ad6ef12d7a5709b
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# kegg-brite-hierarchy-usage

## Summary

Uses KEGG BRITE ko-to-module and ko-to-pathway mapping files, obtained via MetaLAFFA's prepare_databases.py (-km), to aggregate MUSiCC-corrected KO abundances into module- and pathway-level functional profiles with EMPANADA.

## When to use

Use when you have a corrected KEGG orthology group (KO) abundance table and need higher-level KEGG module and pathway abundance profiles, or need to interpret KOs within the KEGG BRITE hierarchy.

## When NOT to use

- Input is raw FASTQ reads or uncorrected gene counts — the aggregation step expects the MUSiCC-corrected KO table, not upstream read-processing outputs
- Functional categories outside the KEGG BRITE hierarchy (e.g. CARD antibiotic resistance genes or UniRef90-based groupings) — the ko-to-module/ko-to-pathway mappings only cover KEGG modules and pathways
- Assembly- or contig-based functional annotation rather than read-based KO profiles

## Inputs

- MUSiCC-corrected KO abundance table (TSV) from MetaLAFFA's ortholog abundance correction step
- KEGG ko-to-module mapping file (downloaded via prepare_databases.py -km)
- KEGG ko-to-pathway mapping file (downloaded via prepare_databases.py -km)

## Outputs

- KEGG module-level abundance table (TSV) in output/final
- KEGG pathway-level abundance table (TSV) in output/final
- Final KO-level abundance table (TSV) alongside the aggregated profiles

## How to apply

Download the KEGG ko-to-module and ko-to-pathway mapping files with prepare_databases.py -km; these mappings define KO membership in higher-level functional categories from the KEGG BRITE hierarchy. Load the MUSiCC-corrected KO abundance table produced by MetaLAFFA's ortholog abundance correction step (inter-sample normalization with intra-sample correction). Run the ortholog_aggregation step (configured in config/steps/ortholog_aggregation.py, orchestrated by Snakemake), which invokes EMPANADA to distribute each KO's abundance among all functional categories it belongs to, weighted by the support for the presence of each category. The aggregated module- and pathway-level abundance tables are written alongside the corrected KO table to the pipeline's final_output_directory (output/final) as tab-separated files.

## Related tools

- **MetaLAFFA** (Pipeline providing the ortholog_aggregation step and final_output_directory structure for the aggregated profiles) — https://github.com/borenstein-lab/MetaLAFFA
- **EMPANADA** (Performs the KO-to-module/pathway aggregation, weighting each KO's abundance by support for the presence of each functional category)
- **Snakemake** (Orchestrates the ortholog_aggregation step in the MetaLAFFA pipeline) — https://github.com/borenstein-lab/MetaLAFFA
- **prepare_databases.py** (Downloads the KEGG ko-to-module and ko-to-pathway mappings via the -km option) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
prepare_databases.py -km -u -hr
```

## Evaluation signals

- output/final contains non-empty TSV tables for KO, module, and pathway abundances (file_exists and file_format_is TSV)
- Aggregated tables have non-zero row counts and contain valid KEGG functional identifiers (module and pathway IDs)
- Each KO's abundance is distributed across all functional categories it belongs to, weighted by category presence support (verify via EMPANADA weighting logic)
- The corrected KO table is reproduced unchanged in the final output alongside the new module and pathway tables
- prepare_databases.py -km completed, producing the ko-to-module and ko-to-pathway mapping files before aggregation runs

## Limitations

- Downloading and preparing reference databases is time and resource intensive: several hours, ~40GB of RAM, and ~300GB of free disk space without cleanup (~150GB with -c cleanup)
- A KO belonging to multiple pathways/modules has its abundance distributed among all such categories, so per-category abundances are support-weighted rather than exclusive counts
- If prepare_databases.py is interrupted and -c was used, the entire database preparation may need to be restarted

## Evidence

- [readme] KEGG ko-to-module and ko-to-pathway mappings: "`-km`: Download KEGG ko-to-module and ko-to-pathway mappings."
- [readme] Aggregating orthology groups into higher-level functional descriptions: "9. Aggregating orthology groups into higher-level functional descriptions"
- [other] EMPANADA aggregation of KO abundances into pathway and module profiles: [evidence span withheld — anchor status: not_found]
- [other] Weighted distribution of KO abundance among categories: [evidence span withheld — anchor status: not_found]
- [other] Final outputs written as TSV in output/final: [evidence span withheld — anchor status: not_found]
