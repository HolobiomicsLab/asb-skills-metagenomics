---
name: kegg-orthology-annotation
description: Use when you have a per-sample gene abundance table and a many-to-many
  gene-to-KEGG-KO (or other ortholog) mapping file, and need a KO-level functional
  profile without double-counting multiply-annotated genes.
when_to_use_negative:
- Input is already an ortholog-level abundance table — the correction would re-split
  already aggregated counts.
- You need reads-to-ortholog annotation from scratch — this step operates downstream
  of read mapping (DIAMOND to UniRef90) and gene counting; run those first.
- Your mapping is one-to-one gene-to-ortholog only and you want unadjusted summed
  counts rather than fractional correction.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3174
tools:
- name: MetaLAFFA
  role: Pipeline providing the ortholog counting step that maps gene abundances to
    KOs with fractional counting
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom Python script performs gene length normalization and fractional ortholog
    counting
- name: Snakemake
  role: Workflow engine executing the ortholog counting step within the MetaLAFFA
    pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:4a79ef90e126440e695bca451e83ddc19a0e065fba98c7a4e4c65157e938b5f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# kegg-orthology-annotation

## Summary

Reconstructs MetaLAFFA's ortholog counting step, which converts per-sample gene abundance tables (gene-length normalized *.norm) into KEGG orthology group (KO) abundances by fractionally splitting each gene's abundance across all KOs it maps to, avoiding double-counting in many-to-many gene-to-ortholog mappings.

## When to use

Use when you have a per-sample gene abundance table and a many-to-many gene-to-KEGG-KO (or other ortholog) mapping file, and need a KO-level functional profile without double-counting multiply-annotated genes.

## When NOT to use

- Input is already an ortholog-level abundance table — the correction would re-split already aggregated counts.
- You need reads-to-ortholog annotation from scratch — this step operates downstream of read mapping (DIAMOND to UniRef90) and gene counting; run those first.
- Your mapping is one-to-one gene-to-ortholog only and you want unadjusted summed counts rather than fractional correction.

## Inputs

- Per-sample gene abundance table (gene-length normalized *.norm TSV from the gene counting step)
- Many-to-many gene-to-ortholog mapping file (TSV, ortholog ID column 1, gene ID column 2) at gene_to_ortholog_maps/<database>_to_<ortholog>.map

## Outputs

- KO abundance table ortholog_abundances.tsv (rows = KO IDs, columns = samples)

## How to apply

Load the gene-length-normalized gene abundance table (*.norm) produced by MetaLAFFA's gene counting step. Load the many-to-many mapping file placed at gene_to_ortholog_maps/<database>_to_<ortholog>.map (tab-separated: ortholog ID in column 1, gene ID in column 2). For each gene, distribute its abundance fractionally across all KOs it is annotated to, so each gene contributes a total of 1 unit of abundance divided evenly among its associated KOs — the fractional contribution method that prevents double-counting. Sum fractional contributions per KO per sample using MetaLAFFA's custom Python counting script run as the ortholog counting step of the Snakemake workflow, writing the corrected-input KO abundance table as ortholog_abundances.tsv (KO IDs as rows, samples as columns).

## Related tools

- **MetaLAFFA** (Pipeline providing the ortholog counting step that maps gene abundances to KOs with fractional counting) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom Python script performs gene length normalization and fractional ortholog counting)
- **Snakemake** (Workflow engine executing the ortholog counting step within the MetaLAFFA pipeline)

## Examples

```
prepare_databases.py -hr -km -u
```

## Evaluation signals

- Per-sample KO abundance totals are conserved relative to input gene abundance totals (each gene contributes exactly 1 unit split across its KOs, no double-counting).
- ortholog_abundances.tsv is a well-formed TSV with KO IDs as rows and samples as columns.
- Genes associated with multiple KOs contribute fractional (summing to 1), not full, abundance to each.
- All KOs present in the mapping file for annotated genes are represented (or accounted for) in the output table.

## Limitations

- Fractional counting assumes the mapping is many-to-many; genes without functional annotations are dropped from KO-level abundances.
- The step requires a precomputed gene-to-ortholog map (e.g., UniRef90-to-KO); preparing default reference databases is resource intensive (~40GB RAM, ~150–300GB disk).
- KO abundances here are corrected-input values; further normalization/correction of orthology group abundances is a separate downstream step.

## Evidence

- [methods] Fractional gene counting for multi-annotations: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [intro] Functional annotation phase role: "the functional annotation phase translates these gene abundances into classifications of community functional capacities"
- [readme] Pipeline overview of orthology mapping: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups"
- [readme] Databases for KO mapping: "`-km`: Download KEGG ko-to-module and ko-to-pathway mappings."
