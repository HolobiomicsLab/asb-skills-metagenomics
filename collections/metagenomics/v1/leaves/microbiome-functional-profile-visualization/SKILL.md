---
name: microbiome-functional-profile-visualization
description: Use when you have a DA results table from pathway_daa() applied to PICRUSt2
  KEGG pathway abundances and need a publication-quality figure showing which pathways
  differ between groups and by how much.
when_to_use_negative:
- When the goal is single-feature pathway significance testing rather than visualization
  of existing DA results — use pathway_daa() first
- When pathway-set-level enrichment (GSEA) is more appropriate than per-pathway error-bar
  plots of DA results — use pathway_gsea()/visualize_gsea() instead
- When the input is still raw KO-level abundance that has not been converted to pathway-level
  abundance (use ko2kegg_abundance() first)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ggpicrust2
  role: Provides pathway_errorbar() and related DA visualization functions (pathway_volcano,
    pathway_heatmap, pathway_pca) for PICRUSt2 functional profiles
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment in which ggpicrust2 plotting functions are executed
  repo: https://github.com/cafferychen777/ggpicrust2
- name: ggplot2
  role: Underlying graphics grammar used by ggpicrust2's visualization functions for
    styling the error-bar plots
  repo: https://github.com/cafferychen777/ggpicrust2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:69daf054f9fd2186a979a6da098f034cf2cf4f400803c453c0a5b60ee2009769
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - R
  - ggpicrust2
  - ggplot2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# microbiome-functional-profile-visualization

## Summary

Visualize differential abundance (DA) results for PICRUSt2-predicted functional profiles using ggpicrust2's ggplot2-based plotting functions, chiefly pathway_errorbar(), which renders group mean relative abundances with error bars annotated with log2 fold change and P-values for differential KEGG pathways.

## When to use

Use when you have a DA results table from pathway_daa() applied to PICRUSt2 KEGG pathway abundances and need a publication-quality figure showing which pathways differ between groups and by how much.

## When NOT to use

- When the goal is single-feature pathway significance testing rather than visualization of existing DA results — use pathway_daa() first
- When pathway-set-level enrichment (GSEA) is more appropriate than per-pathway error-bar plots of DA results — use pathway_gsea()/visualize_gsea() instead
- When the input is still raw KO-level abundance that has not been converted to pathway-level abundance (use ko2kegg_abundance() first)

## Inputs

- KEGG pathway-level abundance table (converted from KO abundances, e.g., ko_abundance)
- Differential abundance results table from pathway_daa() (with log2 fold change and P-values)
- Sample metadata table with group labels

## Outputs

- pathway_errorbar.png: error-bar figure of group mean relative abundances for differential pathways with per-pathway log2 fold change and P-value annotations
- Optionally, companion ggplot2 figures such as volcano plots (pathway_volcano) or heatmaps of the same DA results

## How to apply

Load the KEGG pathway-level abundance table (e.g., the bundled ko_abundance dataset) and the sample metadata in R, then load the annotated DA results table produced by ggpicrust2::pathway_daa(). Call ggpicrust2::pathway_errorbar(), passing the pathway abundance table, the DA results, and the metadata, so the function draws group mean relative abundances with error bars for the differential pathways and annotates each pathway with its log2 fold change and P-value. Styling is ggplot2-based and can be further adjusted with companion packages like cowplot and ggprism before saving the figure (e.g., as pathway_errorbar.png). For complementary views of the same DA results, ggpicrust2 also provides pathway_volcano() for volcano plots and pathway_heatmap(), and compare_metagenome_results() when contrasting DA methods. The rationale is that ggpicrust2 was built specifically to visualize DA results on PICRUSt2 output, including pathway name/description annotation so figures show interpretable pathway labels rather than raw KO identifiers.

## Related tools

- **ggpicrust2** (Provides pathway_errorbar() and related DA visualization functions (pathway_volcano, pathway_heatmap, pathway_pca) for PICRUSt2 functional profiles) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment in which ggpicrust2 plotting functions are executed) — https://github.com/cafferychen777/ggpicrust2
- **ggplot2** (Underlying graphics grammar used by ggpicrust2's visualization functions for styling the error-bar plots) — https://github.com/cafferychen777/ggpicrust2

## Examples

```
p <- pathway_errorbar(ko_abundance, daa_results, metadata); ggsave("pathway_errorbar.png", p)
```

## Evaluation signals

- pathway_errorbar.png exists and is a valid, non-empty figure file
- The figure shows group mean relative abundances with error bars for the differential pathways
- Each plotted pathway is annotated with its log2 fold change and P-value matching the DA results table
- Pathway labels in the figure match annotated pathway names/descriptions rather than raw KO identifiers
- Expert/LLM visual check confirms the error-bar plot faithfully represents the input DA results

## Limitations

- No executable code example with explicit pathway_errorbar parameters was shown in the article's discussion section, so exact argument names may need to be checked against the package documentation
- No software version numbers were stated for ggpicrust2 or PICRUSt2 in the article, so behavior may differ across package versions
- The README notes Bioconductor dependencies (e.g., phyloseq, ALDEx2, ComplexHeatmap) are needed only for certain analysis methods or visualizations, so a minimal CRAN install may lack some plotting backends

## Evidence

- [abstract] Error-bar visualization of DA results: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals."
- [intro] ggpicrust2 scope includes DA visualization: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] Pipeline integration of visualization: [evidence span withheld — anchor status: external_doc]
- [readme] Additional visualization functions: [evidence span withheld — anchor status: not_found]
