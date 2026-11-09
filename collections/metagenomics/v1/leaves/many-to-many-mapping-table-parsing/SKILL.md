---
name: many-to-many-mapping-table-parsing
description: Use when you have a gene-length-normalized gene abundance TSV plus a
  tab-separated gene-to-ortholog map (ortholog ID in column 1, gene ID in column 2)
  and need KO-level functional profiles.
when_to_use_negative:
- Input is already an ortholog-level (e.g. KO-level) abundance table — no mapping-table
  parsing is needed, only downstream normalization or aggregation steps.
- You need reads mapped directly to functions without gene-level intermediates (read-based
  annotation) rather than gene-abundance-based annotation.
- The gene-to-ortholog map is strictly one-to-one, in which case simple summation
  (not fractional splitting across multiple KOs) is appropriate.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaLAFFA
  role: Provides the ortholog counting step that parses the gene-to-KO mapping file
    and computes fractional KO abundances
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Custom Python script performs gene length normalization and fractional counting
    from the mapping table
- name: Snakemake
  role: Wraps the ortholog counting step in the MetaLAFFA workflow so it runs reproducibly
    after gene counting
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:1c95c57805f9af77c59d89e6cbe3fa895bfcbd654b224e1fa1a7fbf7139c4f54
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# many-to-many-mapping-table-parsing

## Summary

Parses a many-to-many gene-to-ortholog mapping table (e.g. UniRef90 genes to KEGG KOs) and converts a per-sample gene abundance table into an ortholog-level abundance table by splitting each gene's abundance fractionally across all orthologs it maps to, avoiding double-counting.

## When to use

Use when you have a gene-length-normalized gene abundance TSV plus a tab-separated gene-to-ortholog map (ortholog ID in column 1, gene ID in column 2) and need KO-level functional profiles.

## When NOT to use

- Input is already an ortholog-level (e.g. KO-level) abundance table — no mapping-table parsing is needed, only downstream normalization or aggregation steps.
- You need reads mapped directly to functions without gene-level intermediates (read-based annotation) rather than gene-abundance-based annotation.
- The gene-to-ortholog map is strictly one-to-one, in which case simple summation (not fractional splitting across multiple KOs) is appropriate.

## Inputs

- per-sample gene abundance table (*.norm, gene-length normalized TSV: genes as rows, samples as columns)
- many-to-many gene-to-ortholog mapping file (gene_to_ortholog_maps/<database>_to_<ortholog>.map, tab-separated, ortholog ID in first column, gene ID in second)

## Outputs

- ortholog_abundances.tsv (corrected-input KO abundance table: KO IDs as rows, samples as columns)

## How to apply

Load the per-sample gene abundance table produced by the gene counting step (the *.norm table, already gene-length normalized by MetaLAFFA's custom Python script). Load the many-to-many mapping file from gene_to_ortholog_maps/<database>_to_<ortholog>.map, which stores ortholog IDs in the first column and gene IDs in the second, so a single gene can appear on multiple lines. For each gene, distribute its abundance fractionally across all orthologs it is annotated to — each gene contributes a total of 1 unit of abundance divided among its associated KOs — then sum the fractional contributions per KO per sample. Run this as the ortholog counting step of the MetaLAFFA Snakemake workflow and write the corrected-input table as ortholog_abundances.tsv; downstream steps can then aggregate KOs into modules and pathways using KEGG ko-to-module and ko-to-pathway mappings fetched by prepare_databases.py -km.

## Related tools

- **MetaLAFFA** (Provides the ortholog counting step that parses the gene-to-KO mapping file and computes fractional KO abundances) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Custom Python script performs gene length normalization and fractional counting from the mapping table)
- **Snakemake** (Wraps the ortholog counting step in the MetaLAFFA workflow so it runs reproducibly after gene counting) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
prepare_databases.py -hr -km -u  # downloads KEGG ko-to-module and ko-to-pathway mappings (-km) and the UniRef90 database (-u) used for gene-to-ortholog counting
```

## Evaluation signals

- Per-sample KO abundance totals are conserved relative to input gene abundance totals — each gene's abundance is split without double-counting across its KOs.
- Every gene ID in the *.norm abundance table that appears in the map contributes to at least one KO; genes absent from the map contribute to no KO row.
- Output ortholog_abundances.tsv is a well-formed TSV with KO IDs as rows and samples as columns, matching the sample column order of the input gene table.
- Each gene's total contribution across all its KOs sums to exactly 1 unit (fraction contribution method), verifiable by re-aggregating KO rows back to genes.
- Downstream aggregation (e.g. KEGG pathways via ko-to-pathway mappings) succeeds on the output, confirming KO IDs are valid identifiers.

## Limitations

- Genes annotated to multiple KOs have their abundance divided evenly, so individual KO abundances for multi-annotated genes are lower bounds on their community contribution.
- The fractional counting and gene length normalization depend on the custom Python script and the *.norm table; skipping gene length normalization biases KO profiles toward long genes.
- Genes without functional annotations are dropped from the KO profile, and the fraction of reads matching UniRef90 can vary widely (53%–78% across samples in the paper), capping coverage of the functional profile.

## Evidence

- [methods] fractional gene counting from the mapping table: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [intro] fractional counts for multi-matches: "If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [readme] pipeline purpose and orthology group abundances: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups."
