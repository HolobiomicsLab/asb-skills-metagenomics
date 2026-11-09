---
name: log2fold-change-pvalue-annotation
description: Use when you have a pathway-level abundance table (e.g. ko_abundance),
  sample metadata, and a DA results table from pathway_daa(), and need to visualize
  effect direction and significance per differential pathway.
when_to_use_negative:
- When the analysis goal is pathway-set enrichment rather than single-pathway DA visualization
  — use pathway_gsea()/visualize_gsea() instead
- When no differential abundance results table exists yet — run pathway_daa() first
- When the input is raw KO abundance without pathway-level conversion or DA testing
  — this skill visualizes DA results, not raw abundances
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ggpicrust2
  role: Provides pathway_errorbar(), the ggplot2-based function that draws error-bar
    plots of group relative abundances annotated with DA log2 fold change and P-values
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment in which the ggpicrust2 visualization is executed
- name: ggplot2
  role: Underlying graphics grammar used by ggpicrust2's visualization layer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:0b993a8d9168b84b7c1a20f391205a6049c3f25dc38ab2e4dacac0469b0e6f13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - R
  - ggpicrust2
  - ggplot2
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# log2fold-change-pvalue-annotation

## Summary

Visualizes PICRUSt2 differential abundance results with ggpicrust2's pathway_errorbar(), rendering group mean relative abundances with error bars for differential KEGG pathways, each annotated with its log2 fold change and P-value from the DA results table.

## When to use

Use when you have a pathway-level abundance table (e.g. ko_abundance), sample metadata, and a DA results table from pathway_daa(), and need to visualize effect direction and significance per differential pathway.

## When NOT to use

- When the analysis goal is pathway-set enrichment rather than single-pathway DA visualization — use pathway_gsea()/visualize_gsea() instead
- When no differential abundance results table exists yet — run pathway_daa() first
- When the input is raw KO abundance without pathway-level conversion or DA testing — this skill visualizes DA results, not raw abundances

## Inputs

- KEGG pathway-level abundance table (e.g. ggpicrust2 built-in data(ko_abundance))
- Sample metadata table (e.g. data(metadata)) with grouping variable
- Differential abundance results table produced by ggpicrust2 pathway_daa(), containing log2 fold change and P-values per pathway

## Outputs

- Error-bar figure (e.g. pathway_errorbar.png) showing group mean relative abundances for differential pathways with log2 fold change and P-value annotations

## How to apply

Load the KEGG pathway-level abundance table (e.g. the built-in ko_abundance dataset) and the sample metadata in R. Load the annotated differential abundance results table produced by pathway_daa(). Call ggpicrust2::pathway_errorbar(), passing the pathway abundance table, the DA results, and the metadata, so the function draws group mean relative abundances with error bars for the differential pathways and annotates each pathway with its log2 fold change and P-value. Apply the ggplot2-based styling of ggpicrust2 and save the rendered figure (e.g. as pathway_errorbar.png). Validation: confirm the file exists, is a valid figure, and shows group error bars with per-pathway log2 fold change and P-value annotations.

## Related tools

- **ggpicrust2** (Provides pathway_errorbar(), the ggplot2-based function that draws error-bar plots of group relative abundances annotated with DA log2 fold change and P-values) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment in which the ggpicrust2 visualization is executed)
- **ggplot2** (Underlying graphics grammar used by ggpicrust2's visualization layer)

## Examples

```
ggpicrust2::pathway_errorbar(kegg_abundance = ko_abundance, daa_results_df = daa_results, metadata = metadata)  # renders error-bar plot with log2 fold change and P-value annotations
```

## Evaluation signals

- pathway_errorbar.png exists on disk and is a valid figure file (file_exists check)
- Expert/LLM visual check confirms the figure shows group error bars for the differential pathways
- Per-pathway log2 fold change and P-value annotations are visible and match the values in the DA results table
- Only pathways present in the supplied DA results table appear in the plot
- The rendered plot reflects the grouping variable defined in the metadata (group means differ accordingly)

## Limitations

- Only 'part of DA results visualization' is integrated in the main ggpicrust2() pipeline; other visualizations (volcano, heatmap, ridgeplot) must be called separately
- Meaningful display depends on pathway name/description annotation quality; reference database updates (v2.1.4: KO reference expanded from 23,917 to 27,531 IDs) affect annotation coverage
- Interpretation of annotated P-values depends on the DA method chosen; the README recommends the camera method as default for more reliable p-values accounting for inter-gene correlations

## Evidence

- [readme] ggpicrust2() integrates ko abundance to kegg pathway abundance conversion, annotation of pathway, differential abundance (DA) analysis, part of DA results visualization.: [evidence span withheld — anchor status: external_doc]
- [intro] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results.: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [abstract] This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals.: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals."
- [readme] camera method (new default): Accounts for inter-gene correlations, providing more reliable p-values than preranked methods (Wu et al., 2012): [evidence span withheld — anchor status: not_found]
