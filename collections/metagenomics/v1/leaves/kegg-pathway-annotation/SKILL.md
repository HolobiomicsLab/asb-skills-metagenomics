---
name: kegg-pathway-annotation
description: Use when a ggpicrust2 differential abundance (DA) results table carries
  KO-level or raw KEGG pathway identifiers and needs pathway_name, pathway_description,
  pathway_class, and pathway_map joined from the KEGG database.
when_to_use_negative:
- The input is a raw KO abundance table or 16S rRNA feature table with no DA statistics
  — annotation operates on DA results, not abundance matrices (use pathway_daa() first).
- The table already carries pathway_name/description/class fields — re-annotating
  adds redundant KEGG REST queries without changing the DA statistics.
- The analysis goal is differential abundance testing itself (e.g. choosing between
  camera, ALDEx2, DESeq2, Maaslin2 methods) rather than interpreting or labeling existing
  DA output.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3308
tools:
- name: ggpicrust2
  role: Provides pathway_annotation(), which queries the KEGG database and joins pathway_name,
    pathway_description, pathway_class, and pathway_map onto DA results; also supplies
    the downstream pathway_errorbar() visualization that consumes the annotated table
  repo: https://github.com/cafferychen777/ggpicrust2
- name: KEGGREST
  role: Optional Bioconductor dependency ('R Interface to KEGG REST API') supporting
    the online KEGG database queries behind pathway annotation
- name: R
  role: Runtime environment in which pathway_annotation() is executed on the DA results
    table
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:d4bc148b2f2476dd7ef25c1e1f25159c604c77cf133ca22718a4305ef8fbd5ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - KEGGREST
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kegg-pathway-annotation

## Summary

ggpicrust2::pathway_annotation() enriches PICRUSt2 differential abundance (DA) results with KEGG reference annotation — pathway_name, pathway_description, pathway_class, and pathway_map — optionally converting KO-level feature identifiers to KEGG pathway-level identifiers via ko_to_kegg = TRUE. It turns a bare statistics table into a readable, pathway-level table ready for visualization.

## When to use

Use when a ggpicrust2 differential abundance (DA) results table carries KO-level or raw KEGG pathway identifiers and needs pathway_name, pathway_description, pathway_class, and pathway_map joined from the KEGG database.

## When NOT to use

- The input is a raw KO abundance table or 16S rRNA feature table with no DA statistics — annotation operates on DA results, not abundance matrices (use pathway_daa() first).
- The table already carries pathway_name/description/class fields — re-annotating adds redundant KEGG REST queries without changing the DA statistics.
- The analysis goal is differential abundance testing itself (e.g. choosing between camera, ALDEx2, DESeq2, Maaslin2 methods) rather than interpreting or labeling existing DA output.

## Inputs

- Differential abundance (DA) results table produced by ggpicrust2::pathway_daa() with KO-level or KEGG pathway-level feature identifiers
- ko_to_kegg logical flag (TRUE when identifiers are KO-level and must be converted to KEGG pathway-level identifiers)
- Online access to the KEGG database (via KEGGREST) for reference annotation lookup

## Outputs

- annotated_da_results.csv — DA results table with pathway_name, pathway_description, pathway_class, and pathway_map joined onto each row
- In-memory R data.frame of annotated DA results suitable as input to pathway_errorbar() and other ggpicrust2 visualization functions

## How to apply

Load the DA results table emitted by pathway_daa() into R, then call ggpicrust2::pathway_annotation() with ko_to_kegg = TRUE when the feature identifiers are KO-level, so they are converted to KEGG pathway-level identifiers before annotation. The function pulls from the online KEGG database (KEGGREST, 'R Interface to KEGG REST API', is the supporting Bioconductor dependency) and joins the pathway_name, pathway_description, pathway_class, and pathway_map fields onto each DA result row. If the input table is already pathway-level, conversion is not needed. Write the annotated table to annotated_da_results.csv and verify the join preserved the DA rows. The annotated table is the expected input for downstream ggpicrust2 visualizations such as pathway_errorbar(), which plots group mean relative abundances with log2 fold change and P-value annotations per pathway.

## Related tools

- **ggpicrust2** (Provides pathway_annotation(), which queries the KEGG database and joins pathway_name, pathway_description, pathway_class, and pathway_map onto DA results; also supplies the downstream pathway_errorbar() visualization that consumes the annotated table) — https://github.com/cafferychen777/ggpicrust2
- **KEGGREST** (Optional Bioconductor dependency ('R Interface to KEGG REST API') supporting the online KEGG database queries behind pathway annotation)
- **R** (Runtime environment in which pathway_annotation() is executed on the DA results table)

## Examples

```
annotated_da_results <- ggpicrust2::pathway_annotation(da_results, ko_to_kegg = TRUE)
```

## Evaluation signals

- annotated_da_results.csv exists and is a valid, readable table (file_exists check)
- Every DA result row carries non-empty pathway_name, pathway_description, pathway_class, and pathway_map fields where KEGG coverage allows
- With ko_to_kegg = TRUE, KO-level feature identifiers are replaced by KEGG pathway-level identifiers (row-level identity check against the input DA table)
- Join is one-to-one: number of rows in the annotated table matches the input DA results (no rows dropped or duplicated)
- The annotated table can be passed directly to pathway_errorbar() to produce a figure with per-pathway log2 fold change and P-value annotations

## Limitations

- pathway_annotation() pulls requests to the online KEGG database, so offline or rate-limited environments will fail or return incomplete annotation.
- Annotation coverage depends on the package's reference databases — the README notes KO reference data was expanded to 27,531 unique KO IDs, so some identifiers may still lack pathway_name/description/class entries.
- The step only labels existing DA results; it does not alter p-values, log2 fold changes, or the choice of DA method.

## Evidence

- [intro] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results.: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] ggpicrust2() integrates ko abundance to kegg pathway abundance conversion, annotation of pathway, differential abundance (DA) analysis, part of DA results visualization.: [evidence span withheld — anchor status: external_doc]
- [readme] KO reference data: Updated from 23,917 to 27,531 unique KO IDs (15.4% increase): [evidence span withheld — anchor status: not_found]
