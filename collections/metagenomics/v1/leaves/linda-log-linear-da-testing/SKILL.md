---
name: linda-log-linear-da-testing
description: Use when you have a PICRUSt2 predicted pathway abundance table plus sample
  metadata with a grouping variable and need log-linear-model-based differential abundance
  testing of pathways between comparison groups.
when_to_use_negative:
- The goal is gene-set-level enrichment rather than single-pathway testing — use pathway_gsea()
  (camera/fry) instead
- You need ANCOM or ANCOMBC — ggpicrust2's pathway_daa() explicitly excludes these
  methods
- Input is raw KO-level abundance without prior conversion to pathway-level abundance
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0605
tools:
- name: ggpicrust2
  role: Provides pathway_daa() which runs LinDA differential abundance testing on
    PICRUSt2 predicted pathway profiles
  repo: https://github.com/cafferychen777/ggpicrust2
- name: LinDA
  role: Log-linear model-based differential abundance method invoked through pathway_daa(daa_method='LinDA')
- name: R
  role: Execution environment for running ggpicrust2 and exporting tidy CSV results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:24c6ca9a823cf7743b542096c23f65c096af0d7b04ae34790c315afff68734a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - LinDA
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# linda-log-linear-da-testing

## Summary

Apply LinDA (log-linear model-based differential abundance) to PICRUSt2-predicted KEGG pathway abundance tables via ggpicrust2's pathway_daa(), producing per-pathway p-values and adjusted p-values for group comparisons.

## When to use

Use when you have a PICRUSt2 predicted pathway abundance table plus sample metadata with a grouping variable and need log-linear-model-based differential abundance testing of pathways between comparison groups.

## When NOT to use

- The goal is gene-set-level enrichment rather than single-pathway testing — use pathway_gsea() (camera/fry) instead
- You need ANCOM or ANCOMBC — ggpicrust2's pathway_daa() explicitly excludes these methods
- Input is raw KO-level abundance without prior conversion to pathway-level abundance

## Inputs

- PICRUSt2-predicted KEGG pathway abundance table (data frame, pathways x samples, e.g. converted from ko_abundance)
- Sample metadata data frame with a categorical grouping variable (e.g. bundled metadata)
- Name of the grouping column in the metadata

## Outputs

- Data frame of DA results with columns feature, method (LinDA), p_val, p_adjust
- Per-method CSV artifact such as daa_results_linda.csv

## How to apply

Load the KEGG pathway abundance table (converted from KO abundances, e.g. via ko2kegg_abundance()) and the bundled sample metadata in R with ggpicrust2 installed. Invoke pathway_daa() with daa_method set to LinDA, supplying the abundance table, metadata, and the grouping variable; LinDA fits log-linear models per pathway to test differential abundance between groups. Convert the returned result into a tidy table with feature, method, p_val, and p_adjust columns and export it as a per-method CSV artifact (e.g. daa_results_linda.csv). LinDA is one of the advanced DA methods ggpicrust2 integrates; the paper notes the package explicitly excludes ANCOM and ANCOMBC, so LinDA is a practical choice when composition-aware methods like ALDEx2 are not required or as a comparison method for robustness checks.

## Related tools

- **ggpicrust2** (Provides pathway_daa() which runs LinDA differential abundance testing on PICRUSt2 predicted pathway profiles) — https://github.com/cafferychen777/ggpicrust2
- **LinDA** (Log-linear model-based differential abundance method invoked through pathway_daa(daa_method='LinDA'))
- **R** (Execution environment for running ggpicrust2 and exporting tidy CSV results)

## Examples

```
library(ggpicrust2); daa_results <- pathway_daa(abundance = kegg_pathway_abundance, metadata = metadata, group = "environment", daa_method = "LinDA")
```

## Evaluation signals

- Output CSV exists, is parseable, and contains the feature, method, p_val, and p_adjust columns with non-empty rows
- Every row in the method column records LinDA, and feature IDs match pathway identifiers from the input abundance table
- p_adjust values are monotonically consistent with p_val (p_adjust >= p_val) and fall in [0,1]
- Row count does not exceed the number of pathways in the input abundance table

## Limitations

- LinDA is a single-feature pathway test; it does not account for inter-gene correlations the way limma camera/fry GSEA does
- pathway_daa() integrates many DA methods but explicitly excludes ANCOM and ANCOMBC
- Results should ideally be compared across methods — ggpicrust2 offers compare_daa_results()/compare_metagenome_results() to assess agreement and robustness

## Evidence

- [other] pathway_daa() is the mechanism for performing differential abundance analysis on PICRUSt2 predicted functional profiles; it integrates nearly all applicable DA methods — including ALDEx2 and LinDA: [evidence span withheld — anchor status: not_found]
- [intro] ggpicrust2 offers advanced DA methods and visualization: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] README lists ALDEx2 as a differential abundance analysis package: [evidence span withheld — anchor status: external_doc]
- [readme] README describes assessing agreement across different DA methods: "compare the consistency and inconsistency across different DA methods applied to the same dataset"
