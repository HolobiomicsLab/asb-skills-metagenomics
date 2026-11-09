---
name: metagenomic-gene-to-ortholog-mapping
description: Use when you have a gene-level (e.g. UniRef90-based, length-normalized)
  abundance table and a gene-to-KO mapping file, and need a KO-level functional profile
  per sample.
when_to_use_negative:
- Input is already an ortholog-level abundance table — aggregating again would distort
  abundances.
- The gene catalog has no functional annotation / gene-to-ortholog mapping available
  (MetaLAFFA's best-hit filtering keeps genes without annotations, which cannot be
  counted at KO level).
- You need pathway-level profiles directly; this step produces KO abundances, and
  pathway aggregation (e.g. via KEGG ko-to-pathway mappings) is a separate downstream
  step.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: MetaLAFFA
  role: 'Pipeline providing the ortholog counting step: fractional KO counting from
    gene abundances via a custom Python script'
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Language of the custom fractional-counting script that performs gene-to-ortholog
    aggregation
- name: Snakemake
  role: Workflow engine that executes the ortholog counting step as part of the MetaLAFFA
    pipeline
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: UniRef90
  role: Reference protein database whose annotations supply the gene-to-ortholog associations
    used for KO mapping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:d00917d6265de13ef0f8a490eae9f8d26907b0714ccd17565263486acca37735
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Python
  - Snakemake
  - UniRef90
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# metagenomic-gene-to-ortholog-mapping

## Summary

Reconstructs KEGG orthology group (KO) abundances from per-sample gene abundance tables using a many-to-many gene-to-ortholog map, splitting each gene's abundance fractionally across all KOs it maps to so total abundance is conserved without double-counting.

## When to use

Use when you have a gene-level (e.g. UniRef90-based, length-normalized) abundance table and a gene-to-KO mapping file, and need a KO-level functional profile per sample.

## When NOT to use

- Input is already an ortholog-level abundance table — aggregating again would distort abundances.
- The gene catalog has no functional annotation / gene-to-ortholog mapping available (MetaLAFFA's best-hit filtering keeps genes without annotations, which cannot be counted at KO level).
- You need pathway-level profiles directly; this step produces KO abundances, and pathway aggregation (e.g. via KEGG ko-to-pathway mappings) is a separate downstream step.

## Inputs

- per-sample gene abundance table (length-normalized *.norm TSV from the gene counting step)
- gene-to-ortholog mapping file (tab-separated, ortholog ID in first column, gene ID in second; gene_to_ortholog_maps/<database>_to_<ortholog>.map)

## Outputs

- ortholog_abundances.tsv — per-sample KO abundance table (KO IDs as rows, samples as columns)

## How to apply

Load the per-sample gene abundance table produced by the gene counting step (the *.norm table, already gene-length normalized by the custom Python script). Load the tab-separated many-to-many gene-to-ortholog mapping in gene_to_ortholog_maps/<database>_to_<ortholog>.map (ortholog ID in column 1, gene ID in column 2). For each gene, distribute its abundance fractionally across all annotated orthology groups so each gene contributes exactly one unit of abundance divided among its KOs, avoiding double-counting. Sum fractional contributions per KO within each sample using MetaLAFFA's Python counting script, executed as the ortholog counting step of the Snakemake workflow, and write the result as ortholog_abundances.tsv (KO IDs as rows, samples as columns).

## Related tools

- **MetaLAFFA** (Pipeline providing the ortholog counting step: fractional KO counting from gene abundances via a custom Python script) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Language of the custom fractional-counting script that performs gene-to-ortholog aggregation)
- **Snakemake** (Workflow engine that executes the ortholog counting step as part of the MetaLAFFA pipeline) — https://github.com/borenstein-lab/MetaLAFFA
- **UniRef90** (Reference protein database whose annotations supply the gene-to-ortholog associations used for KO mapping)

## Examples

```
snakemake output/ortholog_abundances.tsv  # ortholog counting step of the MetaLAFFA Snakemake workflow
```

## Evaluation signals

- Per-sample totals are conserved: each gene contributes exactly 1 unit of abundance divided among its KOs, so summed KO abundances match the input gene abundance totals (no double-counting).
- Output ortholog_abundances.tsv is a well-formed TSV with KO IDs as rows and samples as columns.
- Every KO present in the output traces back to at least one gene in the gene-to-ortholog map and the gene abundance table.
- Genes annotated to multiple KOs appear with fractional (split) contributions rather than full counts in each KO.

## Limitations

- Genes lacking any ortholog annotation (allowed through MetaLAFFA's best-hit filtering) are dropped from the KO-level profile, so KO totals can be lower than gene totals.
- Fractional splitting of multi-annotated genes reduces per-KO magnitudes relative to full-count assignment; downstream normalization (e.g. MUSiCC) may be needed.
- The many-to-many mapping must be placed at the exact path gene_to_ortholog_maps/<database>_to_<ortholog>.map expected by the Snakemake workflow.

## Evidence

- [methods] fractional gene counting method avoids double-counting: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [intro] functional annotation phase translates gene abundances into functional classifications: "the functional annotation phase translates these gene abundances into classifications of community functional capacities"
- [readme] README lists the pipeline step sequence including gene-to-ortholog mapping: "4.  Mapping reads to genes
5.  Mapping genes to functional orthology groups (e.g. KOs)
6.  Aggregating orthologs into higher-level groupings (e.g. pathways)"
- [readme] MetaLAFFA annotates shotgun metagenomic data with orthology group abundances: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups."
