---
name: aldex2-compositional-da-testing
description: Use when you have a PICRUSt2-predicted pathway-level (KEGG) abundance
  table plus sample metadata with a grouping variable, and need to test pathway-level
  differential abundance between comparison groups.
when_to_use_negative:
- When the goal is gene-set-level testing rather than single-feature testing — the
  vignette recommends GSEA (pathway_gsea()) instead of DAA in that case
- When the input is raw KO-level abundance and pathway-level abundances have not yet
  been generated (convert with ko2kegg_abundance() first)
- When ANCOM or ANCOMBC is specifically required — ggpicrust2's pathway_daa() explicitly
  excludes those methods
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3170
tools:
- name: ggpicrust2
  role: R package whose pathway_daa() function orchestrates differential abundance
    testing on PICRUSt2-predicted pathway abundances
  repo: https://github.com/cafferychen777/ggpicrust2
- name: ALDEx2
  role: Optional Bioconductor DA method invoked through pathway_daa() for compositional
    differential abundance testing
- name: LinDA
  role: Alternative DA method in pathway_daa() applying log-linear model-based testing
    for method-consistency comparison
- name: BiocManager
  role: Used to install optional Bioconductor dependencies such as ALDEx2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:54d6bffd9fc8fd41cb76c03ca93e988d32f55e7c6d53a12cbd8eba22979f2a4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - ALDEx2
  - BiocManager
  - LinDA
  - R
  - ggpicrust2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# aldex2-compositional-da-testing

## Summary

Run differential abundance (DA) testing on PICRUSt2-predicted KEGG pathway abundance tables using ggpicrust2's pathway_daa() with the ALDEx2 compositional method (and comparably LinDA), converting results to tidy per-method CSV tables.

## When to use

Use when you have a PICRUSt2-predicted pathway-level (KEGG) abundance table plus sample metadata with a grouping variable, and need to test pathway-level differential abundance between comparison groups.

## When NOT to use

- When the goal is gene-set-level testing rather than single-feature testing — the vignette recommends GSEA (pathway_gsea()) instead of DAA in that case
- When the input is raw KO-level abundance and pathway-level abundances have not yet been generated (convert with ko2kegg_abundance() first)
- When ANCOM or ANCOMBC is specifically required — ggpicrust2's pathway_daa() explicitly excludes those methods

## Inputs

- KEGG pathway-level abundance table derived from PICRUSt2 output (e.g. ko_abundance converted via ko2kegg_abundance())
- Sample metadata table with a categorical grouping variable (ggpicrust2 bundled `metadata` dataset)

## Outputs

- DA results table from pathway_daa() per method (ALDEx2, LinDA)
- Tidy per-method CSV artifacts with columns feature, method, p_val, p_adjust (daa_results_aldex2.csv, daa_results_linda.csv)

## How to apply

Load the KEGG pathway abundance table (e.g. converted from KO abundances via ko2kegg_abundance()) and the bundled metadata in R with ggpicrust2 installed. Invoke pathway_daa() supplying the pathway abundance table, metadata, and grouping variable, setting daa_method = 'ALDEx2' for compositional, centered-log-ratio-based DA testing; repeat with 'LinDA' for a log-linear model-based comparison, noting that ggpicrust2 intentionally excludes ANCOM and ANCOMBC. The package explicitly supports running multiple DA methods on the same dataset so consistency across methods can be assessed as a robustness check. Convert each method's returned object into a tidy table with feature, method, p_val, and p_adjust columns and export one CSV per method (e.g. daa_results_aldex2.csv, daa_results_linda.csv).

## Related tools

- **ggpicrust2** (R package whose pathway_daa() function orchestrates differential abundance testing on PICRUSt2-predicted pathway abundances) — https://github.com/cafferychen777/ggpicrust2
- **ALDEx2** (Optional Bioconductor DA method invoked through pathway_daa() for compositional differential abundance testing)
- **LinDA** (Alternative DA method in pathway_daa() applying log-linear model-based testing for method-consistency comparison)
- **BiocManager** (Used to install optional Bioconductor dependencies such as ALDEx2)

## Examples

```
daa_aldex2 <- pathway_daa(abundance = ko_abundance, metadata = metadata, group = "group", daa_method = "ALDEx2")
```

## Evaluation signals

- Each per-method output CSV (daa_results_aldex2.csv, daa_results_linda.csv) exists and is parseable
- Each CSV contains the columns feature, method, p_val, and p_adjust with non-empty rows
- The method column correctly records which DA method produced each row (ALDEx2 vs LinDA)
- p_adjust values fall in [0,1] and per-feature p_adjust >= p_val
- Running both methods on identical inputs allows consistency assessment across methods as recommended for robustness evaluation

## Limitations

- pathway_daa() integrates nearly all applicable DA methods but explicitly excludes ANCOM and ANCOMBC
- ALDEx2 is an optional Bioconductor dependency that must be installed via BiocManager before the method can be used
- No software version numbers are stated in the article for ggpicrust2 or its DA backends

## Evidence

- [abstract] This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals.: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals."
- [intro] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results.: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome of a particular sample.: "This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome of a particular sample."
