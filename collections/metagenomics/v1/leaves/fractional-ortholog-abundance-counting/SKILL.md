---
name: fractional-ortholog-abundance-counting
description: Use when starting from a per-sample, gene-length-normalized gene abundance
  table plus a many-to-many gene-to-ortholog mapping, and the goal is a KO-level functional
  profile.
when_to_use_negative:
- Input is a raw read-to-gene hits table rather than a gene-length-normalized abundance
  table — the gene counting/normalization step must run first
- The ortholog grouping step has not yet been performed and only per-gene functions
  exist — gene-to-ortholog mapping must be generated before aggregating
- A read-level functional profile is needed rather than a gene-level KO abundance
  profile — this skill operates on aggregated gene abundances, not per-read alignments
edam_operation: http://edamontology.org/operation_0234
edam_topics:
- http://edamontology.org/topic_0176
tools:
- name: MetaLAFFA
  role: Provides the ortholog counting step and the custom Python counting script
    that performs fractional KO abundance summation
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Snakemake
  role: Workflow engine that orchestrates the ortholog counting step within the MetaLAFFA
    pipeline
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Python
  role: Implementation language of the custom counting script performing gene length
    normalization and fractional counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:0e3a38ab65b48a8b2c4c21e5bc2e2c822f5c531a37a682e70ab25f810e9e277c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: http://edamontology.org/operation_0234
  edam_topics:
  - http://edamontology.org/topic_0176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# fractional-ortholog-abundance-counting

## Summary

MetaLAFFA's method for converting per-sample gene abundance tables into KEGG orthology group (KO) abundances by distributing each gene's abundance fractionally across all orthology groups it is mapped to, avoiding double-counting for many-to-many gene-to-ortholog assignments.

## When to use

Use when starting from a per-sample, gene-length-normalized gene abundance table plus a many-to-many gene-to-ortholog mapping, and the goal is a KO-level functional profile.

## When NOT to use

- Input is a raw read-to-gene hits table rather than a gene-length-normalized abundance table — the gene counting/normalization step must run first
- The ortholog grouping step has not yet been performed and only per-gene functions exist — gene-to-ortholog mapping must be generated before aggregating
- A read-level functional profile is needed rather than a gene-level KO abundance profile — this skill operates on aggregated gene abundances, not per-read alignments

## Inputs

- Per-sample gene abundance table (*.norm), gene-length normalized, from the gene counting step
- Gene-to-ortholog mapping file: gene_to_ortholog_maps/<database>_to_<ortholog>.map (tab-separated, ortholog ID first column, gene ID second column)
- Gene annotations linking genes to KEGG orthology groups derived from UniRef90 alignments

## Outputs

- ortholog_abundances.tsv: a TSV table with KO IDs as rows and samples as columns of KO abundances

## How to apply

Load the per-sample gene abundance table produced by MetaLAFFA's gene counting step (the *.norm table, already normalized by gene length). Load the tab-separated gene-to-ortholog mapping placed in gene_to_ortholog_maps/<database>_to_<ortholog>.map, with ortholog ID in the first column and gene ID in the second. For each gene, distribute its abundance fractionally across all KOs it is annotated to, so each gene contributes a total of 1 unit of abundance divided among its associated KOs. Sum the fractional contributions per KO across all genes to obtain KO abundances per sample, using MetaLAFFA's custom Python counting script executed as the ortholog counting step of the Snakemake workflow, and write the result as ortholog_abundances.tsv.

## Related tools

- **MetaLAFFA** (Provides the ortholog counting step and the custom Python counting script that performs fractional KO abundance summation) — https://github.com/borenstein-lab/MetaLAFFA
- **Snakemake** (Workflow engine that orchestrates the ortholog counting step within the MetaLAFFA pipeline) — https://github.com/borenstein-lab/MetaLAFFA
- **Python** (Implementation language of the custom counting script performing gene length normalization and fractional counting)

## Evaluation signals

- Per-sample totals of KO abundances are conserved relative to the input gene abundance totals — no double-counting when a gene maps to multiple KOs
- Each gene's abundance is fully distributed: the sum of its fractional contributions across its KOs equals 1 unit of that gene's abundance
- Output ortholog_abundances.tsv is a well-formed TSV with KO IDs as rows and sample IDs as columns
- Every gene in the input abundance table that appears in the mapping file has its abundance accounted for in at least one KO
- Downstream correction of orthology group abundances (step 8) and aggregation into modules/pathways can consume the table without schema errors

## Limitations

- Fractional counting assumes the gene-to-ortholog mapping may be many-to-many; ortholog abundances may still require downstream correction (step 8) before aggregation into higher-level functional descriptions
- The mapping file format is strict: tab-separated with ortholog ID in column 1 and gene ID in column 2; deviations break the counting script
- Default installation pins specific tool versions (e.g. Snakemake 3.13.3 via Conda), which may affect reproducibility across environments

## Evidence

- [methods] Fractional gene counting for multi-matches (methods basis): "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [intro] Fractional counts rationale in functional annotation phase: "gene abundances are calculated by summing up the number of reads that mapped to each gene. If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [intro] Functional annotation phase purpose: "the functional annotation phase translates these gene abundances into classifications of community functional capacities"
- [readme] Pipeline purpose grounding for ortholog abundance profiles: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups."
