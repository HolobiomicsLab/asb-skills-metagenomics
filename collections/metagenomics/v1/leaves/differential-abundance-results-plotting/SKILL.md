---
name: differential-abundance-results-plotting
description: Use when you have a differential abundance results table for KEGG pathways
  from PICRUSt2 output plus the pathway abundance table and sample metadata, and you
  need a publication figure showing between-group differences.
when_to_use_negative:
- Input is a raw KO abundance table with no DA results yet — run differential abundance
  analysis first
- You want GSEA result visuals (enrichment plots, ridge plots) rather than DA error
  bars — use visualize_gsea() or pathway_ridgeplot()
- You need a per-taxa contribution view of which organisms drive pathway shifts —
  use the taxa contribution workflow (taxa_contribution_bar/heatmap) instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: ggpicrust2
  role: provides pathway_errorbar() to render the DA results error-bar figure with
    log2FC and P-value annotations
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: runtime environment in which the ggpicrust2 visualization is executed and
    the figure is saved
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:54a81d324310c4b70d1bb2f295cf39b7bbd321009062556d237d5f7756acf935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# differential-abundance-results-plotting

## Summary

Render error-bar plots of PICRUSt2 predicted pathway differential abundance results with ggpicrust2::pathway_errorbar(), showing group mean relative abundances with error bars annotated with log2 fold change and P-values per pathway.

## When to use

Use when you have a differential abundance results table for KEGG pathways from PICRUSt2 output plus the pathway abundance table and sample metadata, and you need a publication figure showing between-group differences.

## When NOT to use

- Input is a raw KO abundance table with no DA results yet — run differential abundance analysis first
- You want GSEA result visuals (enrichment plots, ridge plots) rather than DA error bars — use visualize_gsea() or pathway_ridgeplot()
- You need a per-taxa contribution view of which organisms drive pathway shifts — use the taxa contribution workflow (taxa_contribution_bar/heatmap) instead

## Inputs

- KEGG pathway-level abundance table (e.g. ko_abundance data object; KO table with ko_to_kegg=TRUE)
- Sample metadata table (e.g. metadata data object)
- Differential abundance results table (from pathway_daa or pathway_annotation step)

## Outputs

- Error-bar figure (e.g. pathway_errorbar.png) showing group mean relative abundances with error bars for differential pathways
- Per-pathway log2 fold change and P-value annotations overlaid on the figure

## How to apply

Load the KEGG pathway-level abundance table (e.g. the packaged ko_abundance data) and sample metadata into R, then load the DA results table produced by the pathway_daa/annotation steps. Call ggpicrust2::pathway_errorbar(), passing the pathway abundance table, the DA results, and the metadata; the function draws group mean relative abundances with error bars for the differential pathways and annotates each pathway with its log2 fold change and P-value. If your features are still KO-level rather than KEGG pathway-level, set ko_to_kegg = TRUE so identifiers are converted before plotting. Styling is ggplot2-based (ggpicrust2 depends on ggplot2, ggprism, cowplot, ggforce), so the rendered figure can be saved directly (e.g. as pathway_errorbar.png) and restyled with ggplot2 syntax.

## Related tools

- **ggpicrust2** (provides pathway_errorbar() to render the DA results error-bar figure with log2FC and P-value annotations) — https://github.com/cafferychen777/ggpicrust2
- **R** (runtime environment in which the ggpicrust2 visualization is executed and the figure is saved)

## Examples

```
data(ko_abundance); data(metadata); pathway_errorbar(ka = ko_abundance, daa_results_table = da_results, metadata = metadata, ko_to_kegg = TRUE)
```

## Evaluation signals

- pathway_errorbar.png exists and is a valid, non-empty image file
- Figure shows group error bars for the differential pathways (visual/expert check)
- Each differential pathway carries log2 fold change and P-value annotations on the plot
- Set of plotted pathways matches the significant rows of the annotated DA results table
- Feature identifiers in the plot are KEGG pathway-level (ko_to_kegg conversion applied if starting from KO IDs)

## Limitations

- pathway_errorbar() is only one part of DA results visualization; ggpicrust2() 'integrates ko abundance to kegg pathway abundance conversion, annotation of pathway, differential abundance (DA) analysis, part of DA results visualization', so some plotting is outside the main pipeline
- If DA results are unannotated, plot labels may lack pathway_name/description until pathway_annotation() has been run
- No software version numbers are stated for ggpicrust2 in the article, so reproducibility of exact styling may depend on the installed release

## Evidence

- [readme] pathway_errorbar renders error-bar visualization: [evidence span withheld — anchor status: external_doc]
- [intro] visualization of DA results is a core feature: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [intro] packaged example abundance and metadata inputs: "data(ko_abundance)
data(metadata)"
- [abstract] extensive DA analyses and visualizations: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals."
