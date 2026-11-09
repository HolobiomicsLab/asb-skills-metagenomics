---
name: functional-profile-table-generation
description: Use when a MetaLAFFA (or similar KO-based) analysis needs corrected KO
  abundances rolled up into higher-level KEGG functional categories and written as
  final TSV functional profile tables.
when_to_use_negative:
- Input is raw FASTQ reads rather than a corrected ortholog abundance table — run
  upstream MetaLAFFA steps (host/duplicate filtering, quality trimming, read mapping,
  ortholog counting, MUSiCC correction) first
- The desired higher-level groupings are not KEGG modules or pathways (e.g. UniRef,
  eggNOG, or CARD-based functional categories), since the ko-to-module/ko-to-pathway
  mappings are KEGG-specific
- Aggregating taxonomic or abundance-normalized feature tables unrelated to functional
  orthologs
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: MetaLAFFA
  role: Pipeline whose ortholog_aggregation step performs EMPANADA-based aggregation
    and writes final functional profile tables
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: EMPANADA
  role: Aggregates KO abundances into KEGG pathway and module abundances weighted
    by support for the presence of each category
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Snakemake
  role: Orchestrates the ortholog_aggregation step and final output generation
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Module-based configuration system (config/steps/ortholog_aggregation.py) parameterizing
    the aggregation
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:b9aac9024141bbd2f9d8a8b614020031fa8a971bd777cb20eba5a14acf8ccabc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - EMPANADA
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# functional-profile-table-generation

## Summary

Aggregate MUSiCC-corrected KEGG ortholog (KO) abundances into KEGG module- and pathway-level functional profiles using EMPANADA, then emit KO-, module-, and pathway-level abundance tables as the pipeline's final outputs.

## When to use

Use when a MetaLAFFA (or similar KO-based) analysis needs corrected KO abundances rolled up into higher-level KEGG functional categories and written as final TSV functional profile tables.

## When NOT to use

- Input is raw FASTQ reads rather than a corrected ortholog abundance table — run upstream MetaLAFFA steps (host/duplicate filtering, quality trimming, read mapping, ortholog counting, MUSiCC correction) first
- The desired higher-level groupings are not KEGG modules or pathways (e.g. UniRef, eggNOG, or CARD-based functional categories), since the ko-to-module/ko-to-pathway mappings are KEGG-specific
- Aggregating taxonomic or abundance-normalized feature tables unrelated to functional orthologs

## Inputs

- MUSiCC-corrected KO abundance table (TSV) from the ortholog abundance correction step
- KEGG ko-to-module mapping file (downloaded via prepare_databases.py -km)
- KEGG ko-to-pathway mapping file (downloaded via prepare_databases.py -km)

## Outputs

- Corrected KO-level abundance table (TSV) in output/final
- Aggregated KEGG module-level abundance table (TSV)
- Aggregated KEGG pathway-level abundance table (TSV)

## How to apply

Load the MUSiCC-corrected KO abundance table produced by the ortholog abundance correction step (inter-sample normalization with intra-sample correction). Load the KEGG ko-to-module and ko-to-pathway mapping files downloaded with prepare_databases.py using the -km flag, which define KO membership per the KEGG BRITE hierarchy. Invoke the MetaLAFFA ortholog_aggregation step (configured in config/steps/ortholog_aggregation.py and orchestrated by Snakemake) to run EMPANADA, which distributes each KO's abundance among all functional categories it belongs to, weighted by the support for the presence of each category. Write the resulting module- and pathway-level abundance tables alongside the corrected KO table as tab-separated files in the pipeline's final_output_directory (output/final).

## Related tools

- **MetaLAFFA** (Pipeline whose ortholog_aggregation step performs EMPANADA-based aggregation and writes final functional profile tables) — https://github.com/borenstein-lab/MetaLAFFA
- **EMPANADA** (Aggregates KO abundances into KEGG pathway and module abundances weighted by support for the presence of each category) — https://github.com/borenstein-lab/MetaLAFFA
- **Snakemake** (Orchestrates the ortholog_aggregation step and final output generation) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Module-based configuration system (config/steps/ortholog_aggregation.py) parameterizing the aggregation) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
prepare_databases.py -km -u -hr  # then run MetaLAFFA's ortholog_aggregation step (Snakemake) to emit KO/module/pathway TSVs in output/final
```

## Evaluation signals

- The final_output_directory (output/final) contains non-empty TSV tables for KO, module, and pathway abundances (file_exists / file_format_is TSV checks pass)
- Each output table has non-zero row counts
- Tables contain valid KEGG functional identifiers (K-numbers for KOs, module M-IDs, and pathway IDs from the KEGG BRITE hierarchy)
- Module and pathway abundance values are consistent with the corrected KO table after EMPANADA's support-weighted distribution
- Summary statistics for the aggregation step report no errors, as MetaLAFFA generates per-step summary statistics

## Limitations

- Aggregation depends on KEGG ko-to-module and ko-to-pathway mappings, so KOs without mappings to a category do not contribute to it
- The prepare_databases.py -km download step is time and resource intensive (several hours, ~40GB RAM, ~150–300GB disk) if run as part of full database preparation
- MUSiCC may throw an 'undefined symbol: omp_get_num_procs' error in some environments, blocking the upstream corrected KO table this step requires

## Evidence

- [readme] EMPANADA aggregation: "9. Aggregating orthology groups into higher-level functional descriptions"
- [readme] KO-to-module/pathway mappings via prepare_databases: "`-km`: Download KEGG ko-to-module and ko-to-pathway mappings."
- [readme] Overview of aggregation into higher-level groupings: "Aggregating orthologs into higher-level groupings (e.g. pathways)"
- [readme] Final outputs documentation: "Final outputs location and descriptions"
- [intro] Functional annotation phase translates gene abundances into functional capacities: "the functional annotation phase translates these gene abundances into classifications of community functional capacities"
- [abstract] MetaLAFFA generates per-step summary statistics: "MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality"
