---
name: r-data-frame-annotation-wrangling
description: Use when you hold a DA results table (feature IDs are KO terms or KEGG
  pathway IDs) from ggpicrust2/PICRUSt2 output and need pathway name, description,
  class, and map fields joined onto each row.
when_to_use_negative:
- Input is already an annotated table with pathway_name/description/class columns
  — re-annotation adds no information
- Feature identifiers are not KO terms or KEGG pathway IDs (e.g. EC numbers or MetaCyc
  pathways) — ko_to_kegg=TRUE conversion would not apply
- You need taxon-level attribution of pathway shifts — use the PICRUSt2 per-sequence
  contribution workflow (read_contrib_file()/read_strat_file()) instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: ggpicrust2
  role: Supplies pathway_annotation(), which converts KO-level identifiers to KEGG
    pathway-level identifiers (ko_to_kegg=TRUE) and pulls pathway_name, pathway_description,
    pathway_class, and pathway_map annotation onto DA result rows
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment for loading the DA results table, running pathway_annotation(),
    and writing the annotated CSV
- name: KEGGREST
  role: Optional Bioconductor dependency listed for KEGG REST API access underlying
    annotation workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:985a43350ab3cfae9e9f5ad414a31354fa8e67256ac64c51ce1d149b0a8a0dbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - KEGGREST
  - R
  - ggpicrust2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-data-frame-annotation-wrangling

## Summary

Annotate a differential abundance (DA) results table from PICRUSt2-predicted functional profiling with KEGG pathway metadata (pathway_name, pathway_description, pathway_class, pathway_map) using ggpicrust2::pathway_annotation() in R, so that DA features identified by KO/pathway IDs become human-interpretable rows.

## When to use

Use when you hold a DA results table (feature IDs are KO terms or KEGG pathway IDs) from ggpicrust2/PICRUSt2 output and need pathway name, description, class, and map fields joined onto each row.

## When NOT to use

- Input is already an annotated table with pathway_name/description/class columns — re-annotation adds no information
- Feature identifiers are not KO terms or KEGG pathway IDs (e.g. EC numbers or MetaCyc pathways) — ko_to_kegg=TRUE conversion would not apply
- You need taxon-level attribution of pathway shifts — use the PICRUSt2 per-sequence contribution workflow (read_contrib_file()/read_strat_file()) instead

## Inputs

- DA results data frame from ggpicrust2::pathway_daa() (rows keyed by KO or KEGG pathway identifiers)
- ko_to_kegg logical parameter (TRUE when input identifiers are KO-level)

## Outputs

- Annotated data frame with pathway_name, pathway_description, pathway_class, and pathway_map columns joined onto each DA result row
- CSV file of annotated DA results (e.g. annotated_da_results.csv)

## How to apply

Load the DA results table emitted by the differential abundance analysis step into R, then call ggpicrust2::pathway_annotation() with ko_to_kegg = TRUE when the feature identifiers are KO-level so they are converted to KEGG pathway-level identifiers. The function queries reference/online KEGG annotation and returns columns pathway_name, pathway_description, pathway_class, and pathway_map, which are joined onto each DA result row. The rationale is that ggpicrust2's updated reference databases (KO reference expanded from 23,917 to 27,531 unique KO IDs in v2.1.4) provide more complete coverage for recently discovered KEGG orthology entries without changing any code. Finally, write the annotated table (e.g. annotated_da_results.csv) for downstream visualization such as pathway_errorbar() or pathway_heatmap(). Note that ggpicrust2() also integrates KO-to-pathway conversion and annotation as part of the main pipeline if a one-call workflow is preferred.

## Related tools

- **ggpicrust2** (Supplies pathway_annotation(), which converts KO-level identifiers to KEGG pathway-level identifiers (ko_to_kegg=TRUE) and pulls pathway_name, pathway_description, pathway_class, and pathway_map annotation onto DA result rows) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment for loading the DA results table, running pathway_annotation(), and writing the annotated CSV)
- **KEGGREST** (Optional Bioconductor dependency listed for KEGG REST API access underlying annotation workflows)

## Examples

```
annotated <- ggpicrust2::pathway_annotation(input = da_results, ko_to_kegg = TRUE)
```

## Evaluation signals

- Annotated output contains the four annotation columns: pathway_name, pathway_description, pathway_class, pathway_map
- Every row's feature identifier is a KEGG pathway-level ID after annotation when ko_to_kegg = TRUE (no raw KO-level IDs remain)
- Row count of the annotated table equals the input DA results table (annotation is a left join, no rows dropped)
- Spot-check a known pathway ID (e.g. ko00970 for photosynthesis-related) against the KEGG reference to confirm name/description/class match
- Written CSV (annotated_da_results.csv) parses cleanly and preserves the annotation columns

## Limitations

- Annotation coverage depends on packaged reference databases; older ggpicrust2 versions had smaller KO/EC references (23,917 KO IDs pre-v2.1.4), so very recently discovered KEGG entries may be missing in outdated installs
- pathway_annotation() may pull requests to the online KEGG database, so results depend on KEGG availability and network access
- No software version numbers are stated for ggpicrust2 in the article, making exact annotation-content reproducibility uncertain across package versions

## Evidence

- [intro] ggpicrust2 aims to provide pathway annotation, DA and visualization features: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] ggpicrust2() integrates KO-to-pathway conversion, annotation, and DA analysis: "ggpicrust2() integrates ko abundance to kegg pathway abundance
conversion, annotation of pathway, differential abundance (DA) analysis,
part of DA results visualization."
- [readme] Updated KO reference database improves annotation coverage: "**KO reference data**: Updated from 23,917 to 27,531 unique KO IDs
  (15.4% increase)"
