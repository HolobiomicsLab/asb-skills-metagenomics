---
name: kegg-reference-database-mapping
description: Use when DA result tables from ggpicrust2::pathway_daa() carry KO-level
  or unannotated feature IDs and need KEGG pathway names, descriptions, and classes
  joined on before visualization or reporting.
when_to_use_negative:
- Input features are already annotated KEGG pathway-level results — re-annotating
  duplicates reference lookup without adding information
- You need GSEA result annotation rather than DA result annotation — use ggpicrust2::gsea_pathway_annotation()
  for GSEA output instead
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3391
tools:
- name: ggpicrust2
  role: Provides pathway_annotation() which maps KO identifiers to KEGG pathways and
    joins pathway_name, pathway_description, pathway_class, and pathway_map onto DA
    results
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment in which the ggpicrust2 annotation function is executed
- name: KEGGREST
  role: Bioconductor interface to the KEGG REST API listed as a dependency supporting
    KEGG reference annotation
- name: BiocManager
  role: Used to install Bioconductor dependencies (e.g. KEGGREST) needed for the annotation
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:a6c7d6eeab7ecd9512a6529cabeaa39141da68a03c31b4abf910a97a8d9d16d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - BiocManager
  - KEGGREST
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kegg-reference-database-mapping

## Summary

Annotate PICRUSt2 differential abundance (DA) results by mapping KO-level feature identifiers to KEGG pathway-level identifiers and joining reference annotation fields (pathway_name, pathway_description, pathway_class, pathway_map) using ggpicrust2::pathway_annotation().

## When to use

Use when DA result tables from ggpicrust2::pathway_daa() carry KO-level or unannotated feature IDs and need KEGG pathway names, descriptions, and classes joined on before visualization or reporting.

## When NOT to use

- Input features are already annotated KEGG pathway-level results — re-annotating duplicates reference lookup without adding information
- You need GSEA result annotation rather than DA result annotation — use ggpicrust2::gsea_pathway_annotation() for GSEA output instead

## Inputs

- Differential abundance results table from ggpicrust2::pathway_daa() with KO-level or pathway-level feature identifiers
- PICRUSt2 predicted functional profile output (e.g. ko abundance table as in the bundled ko_abundance dataset)

## Outputs

- Annotated DA results table with pathway_name, pathway_description, pathway_class, and pathway_map columns (e.g. annotated_da_results.csv)

## How to apply

Load the DA results table produced by ggpicrust2::pathway_daa() into R and call ggpicrust2::pathway_annotation() with ko_to_kegg set to TRUE when the input features are KO-level identifiers, so they are first converted to KEGG pathway-level identifiers. The function pulls KEGG reference annotation and joins the pathway_name, pathway_description, pathway_class, and pathway_map fields onto each DA result row, then write the annotated table (e.g. annotated_da_results.csv). The mapping relies on package-bundled reference data that was expanded in v2.1.4 — KO reference coverage grew from 23,917 to 27,531 unique KO IDs and EC reference entries from 3,180 to 8,371 — improving annotation of recently discovered enzymes and KEGG orthology entries without code changes. KEGGREST is the listed Bioconductor interface supporting the KEGG reference lookup, so install it when setting up dependencies via BiocManager.

## Related tools

- **ggpicrust2** (Provides pathway_annotation() which maps KO identifiers to KEGG pathways and joins pathway_name, pathway_description, pathway_class, and pathway_map onto DA results) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment in which the ggpicrust2 annotation function is executed)
- **KEGGREST** (Bioconductor interface to the KEGG REST API listed as a dependency supporting KEGG reference annotation)
- **BiocManager** (Used to install Bioconductor dependencies (e.g. KEGGREST) needed for the annotation workflow)

## Examples

```
annotated <- ggpicrust2::pathway_annotation(input_da_result_file = "da_results.csv", ko_to_kegg = TRUE); write.csv(annotated, "annotated_da_results.csv")
```

## Evaluation signals

- Every DA result row carries non-missing pathway_name and pathway_description after annotation
- KO-level feature identifiers are replaced by KEGG pathway-level identifiers when ko_to_kegg = TRUE
- Annotated table contains all four reference fields: pathway_name, pathway_description, pathway_class, pathway_map
- Annotation coverage is consistent with the updated reference database (27,531 unique KO IDs since v2.1.4); low coverage suggests stale package versions

## Limitations

- Reference database coverage limits annotation: KO entries not in the bundled 27,531-ID reference and EC entries outside the 8,371-entry EC reference will lack complete annotation
- Version-dependent behavior: pre-v2.1.4 ggpicrust2 installs use smaller reference databases (23,917 KO IDs, 3,180 EC entries), yielding lower annotation coverage
- ko_to_kegg must be set correctly for the input feature type; wrong setting mismatches identifiers against the reference

## Evidence

- [intro] pathway_annotation() fields: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] reference database update: [evidence span withheld — anchor status: external_doc]
- [readme] EC reference update: [evidence span withheld — anchor status: external_doc]
- [readme] KEGGREST dependency: [evidence span withheld — anchor status: external_doc]
- [full_text] ko to pathway conversion workflow: "convert KO profiles to pathway-level abundance when needed"
