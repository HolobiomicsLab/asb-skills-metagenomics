---
name: microbiome-functional-data-interpretation
description: Use when input is PICRUSt2 predicted functional output (e.g., KO abundance
  tables and sample metadata) and the goal is differential pathway abundance, pathway
  annotation, or GSEA between comparison groups.
when_to_use_negative:
- Input is already pathway-level or a non-PICRUSt2 feature table requiring no KO-to-KEGG
  conversion or functional prediction interpretation
- Single-feature differential testing is preferred and pathway-set (GSEA) framing
  is not appropriate for the question
- No comparison-group design exists (no metadata grouping or contrasts), so differential
  abundance or enrichment testing is not applicable
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ggpicrust2
  role: Core R package providing ko2kegg_abundance, pathway_daa, pathway_annotation,
    pathway_gsea, and visualization functions for PICRUSt2 functional profiles
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Execution environment in which ggpicrust2 workflows are run and example datasets
    are loaded
- name: limma
  role: Bioconductor dependency whose camera and fry competitive gene set tests back
    pathway_gsea()
- name: PICRUSt2
  role: Upstream tool that predicts microbiome function from 16S rRNA gene data, producing
    the KO abundance inputs consumed here
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:7c9a96b871e5e02d990780533deae6474bc2de7bc25575d6e214002a4712cfd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - PICRUSt2
  - R
  - ggpicrust2
  - limma
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# microbiome-functional-data-interpretation

## Summary

Interprets PICRUSt2-predicted functional profiles from 16S rRNA sequencing data using the ggpicrust2 R package, covering KO-to-KEGG pathway conversion, differential abundance analysis, pathway annotation, and gene set enrichment analysis (GSEA).

## When to use

Use when input is PICRUSt2 predicted functional output (e.g., KO abundance tables and sample metadata) and the goal is differential pathway abundance, pathway annotation, or GSEA between comparison groups.

## When NOT to use

- Input is already pathway-level or a non-PICRUSt2 feature table requiring no KO-to-KEGG conversion or functional prediction interpretation
- Single-feature differential testing is preferred and pathway-set (GSEA) framing is not appropriate for the question
- No comparison-group design exists (no metadata grouping or contrasts), so differential abundance or enrichment testing is not applicable

## Inputs

- PICRUSt2 KO abundance table (e.g., data(ko_abundance))
- Sample metadata table mapping samples to comparison groups (e.g., data(metadata))
- Differential abundance results table from pathway_daa() for annotation

## Outputs

- KEGG pathway-level abundance table (ko2kegg_abundance output)
- Differential abundance results table with p-value and p.adjust columns
- Annotated DA results with pathway_name, pathway_description, pathway_class, pathway_map (e.g., annotated_da_results.csv)
- GSEA results table with pathway ID, NES, p-value, p.adjust (e.g., gsea_results.csv)
- Visualizations (volcano plots, heatmaps, PCA, GSEA enrichment/dot/network plots)

## How to apply

Load the bundled example data with data(ko_abundance) and data(metadata) in R, or supply your own PICRUSt2 KO abundance table and metadata. Convert KO-level features to KEGG pathway-level abundance with ko2kegg_abundance(), then run pathway_daa() with one or more differential abundance methods, optionally comparing consistency across methods with compare_daa_results(). Annotate results via pathway_annotation() (with ko_to_kegg = TRUE) to join pathway_name, pathway_description, pathway_class, and pathway_map from KEGG. When pathway-set analysis is more appropriate than single-feature testing, run pathway_gsea(); use the camera method (the default) because it accounts for inter-gene correlations and provides more reliable p-values than preranked methods, use fry for faster computation on large gene set collections, and adjust for covariates such as age, sex, or BMI when present. Retain gene sets with sizes between min_size=5 and max_size=500, and visualize results with pathway_volcano(), pathway_heatmap(), or visualize_gsea().

## Related tools

- **ggpicrust2** (Core R package providing ko2kegg_abundance, pathway_daa, pathway_annotation, pathway_gsea, and visualization functions for PICRUSt2 functional profiles) — https://github.com/cafferychen777/ggpicrust2
- **R** (Execution environment in which ggpicrust2 workflows are run and example datasets are loaded)
- **limma** (Bioconductor dependency whose camera and fry competitive gene set tests back pathway_gsea())
- **PICRUSt2** (Upstream tool that predicts microbiome function from 16S rRNA gene data, producing the KO abundance inputs consumed here)

## Examples

```
data(ko_abundance); data(metadata); results <- ggpicrust2::pathway_gsea(ko_abundance, metadata, method = "camera", min_size = 5, max_size = 500)
```

## Evaluation signals

- gsea_results.csv exists, is a valid CSV, and contains pathway ID, NES, p-value, and p.adjust columns with gene set sizes within the 5–500 range
- pathway_annotation output rows carry the KEGG fields pathway_name, pathway_description, pathway_class, and pathway_map
- GSEA is run with method='camera' (the default) rather than a preranked method, per the package's recommendation
- Differential abundance results show consistency or explainable discrepancy when compared across multiple DA methods via compare_daa_results()
- Covariates such as age, sex, or BMI are included in the design when present in metadata

## Limitations

- Preranked GSEA methods can produce 'spectacularly wrong p-values' by not accounting for inter-gene correlations; prefer camera or fry
- No software version numbers are stated for ggpicrust2 or PICRUSt2 in the article, complicating reproducibility pinning
- No seed or randomness control information is documented for the GSEA implementations

## Evidence

- [abstract] We have developed ggpicrust2, an R package, for analyzing functional profiles derived from 16S rRNA sequencing.: "We have developed ggpicrust2, an R package, for analyzing functional profiles derived from 16S rRNA sequencing."
- [abstract] no state-of-art inference tools are available to test the differences in pathway abundances between comparison groups: "no state-of-art inference tools are available to test the differences in pathway abundances between comparison groups."
- [discussion] camera method provides more reliable p-values by accounting for inter-gene correlations: "it provides more reliable p-values by accounting for inter-gene correlations"
- [intro] perform GSEA when pathway-set analysis is more appropriate than single-feature testing: "perform GSEA when pathway-set analysis is more appropriate than single-feature testing"
- [readme] camera method (new default): Accounts for inter-gene correlations, providing more reliable p-values than preranked methods: [evidence span withheld — anchor status: external_doc]
- [discussion] Adjust for covariates when you have confounding factors like age, sex, or BMI in your study design: "**Adjust for covariates** when you have confounding factors like age, sex, or BMI in your study design"
