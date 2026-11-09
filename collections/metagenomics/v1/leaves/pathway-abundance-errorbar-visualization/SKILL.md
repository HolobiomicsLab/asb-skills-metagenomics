---
name: pathway-abundance-errorbar-visualization
description: Use when you already hold a KEGG pathway-level abundance table (e.g.
  ko_abundance), sample metadata, and a ggpicrust2 DA results table, and need a publication-quality
  error-bar figure of differential pathways.
when_to_use_negative:
- You only have raw KO-level abundance and have not yet run pathway-level conversion
  and differential abundance analysis (pathway_daa) — the errorbar plot requires DA
  results as input
- You need gene-set-level enrichment rather than single-pathway DA visualization —
  use the pathway_gsea()/visualize_gsea() workflow instead
- You want to inspect which taxa drive pathway shifts — use the PICRUSt2 per-sequence
  contribution workflow (read_contrib_file(), taxa_contribution_bar()) instead
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ggpicrust2
  role: Provides pathway_errorbar() to render the error-bar visualization of DA results
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment in which the ggpicrust2 plotting workflow is executed
- name: ggplot2
  role: Underlying grammar-of-graphics engine used by pathway_errorbar() for styling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:d439da8672586829912247cdbafe21e362dec9ab4dd844af250622cc6a4cf7c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - R
  - ggpicrust2
  - ggplot2
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pathway-abundance-errorbar-visualization

## Summary

Visualize PICRUSt2 predicted pathway-level differential abundance results with ggpicrust2::pathway_errorbar(), rendering group mean relative abundances with error bars annotated with log2 fold change and P-values for differential pathways.

## When to use

Use when you already hold a KEGG pathway-level abundance table (e.g. ko_abundance), sample metadata, and a ggpicrust2 DA results table, and need a publication-quality error-bar figure of differential pathways.

## When NOT to use

- You only have raw KO-level abundance and have not yet run pathway-level conversion and differential abundance analysis (pathway_daa) — the errorbar plot requires DA results as input
- You need gene-set-level enrichment rather than single-pathway DA visualization — use the pathway_gsea()/visualize_gsea() workflow instead
- You want to inspect which taxa drive pathway shifts — use the PICRUSt2 per-sequence contribution workflow (read_contrib_file(), taxa_contribution_bar()) instead

## Inputs

- KEGG pathway-level abundance table (e.g. ggpicrust2 dataset ko_abundance, samples x pathways)
- Sample metadata table (e.g. ggpicrust2 dataset metadata) with group labels
- Differential abundance results data frame produced by ggpicrust2::pathway_daa() (pathway, log2 fold change, P-value per method)

## Outputs

- Error-bar figure (pathway_errorbar.png) showing group mean relative abundances with error bars for differential pathways, annotated with log2 fold change and P-values

## How to apply

In R, load the KEGG pathway-level abundance table (e.g. the package dataset ko_abundance) and the sample metadata, then load the differential abundance results table produced upstream by ggpicrust2::pathway_daa() (optionally annotated via pathway_annotation()). Call ggpicrust2::pathway_errorbar(), passing the pathway abundance table, the DA results, and the metadata, so the function draws group mean relative abundances with error bars for the differential pathways and annotates each pathway with its log2 fold change and P-value. The plotting is built on ggplot2 and the package's ggprism/cowplot-based styling, so the output is a publication-quality figure; save it as pathway_errorbar.png. Rationale: ggpicrust2 was built specifically to add visualization of DA results for PICRUSt2 functional profiles, since no state-of-the-art inference/visualization tools previously existed for testing pathway abundance differences between comparison groups.

## Related tools

- **ggpicrust2** (Provides pathway_errorbar() to render the error-bar visualization of DA results) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment in which the ggpicrust2 plotting workflow is executed)
- **ggplot2** (Underlying grammar-of-graphics engine used by pathway_errorbar() for styling)

## Examples

```
pathway_errorbar(abundance = ko_abundance, daa_results_df = daa_results, metadata = metadata, Group = "Group")
```

## Evaluation signals

- pathway_errorbar.png exists on disk and is a valid figure file
- Expert/LLM visual check confirms the figure shows group mean relative abundances with error bars for the differential pathways
- Each plotted pathway carries its log2 fold change and P-value annotation matching the DA results table
- Plotted pathways correspond to rows present in both the pathway abundance table and the DA results table
- Figure styling matches ggpicrust2's ggplot2-based publication-quality theme (ggprism/cowplot dependencies)

## Limitations

- Requires a completed upstream DA step (pathway_daa results); it cannot visualize differential pathways from abundance data alone
- No software version numbers are stated for ggpicrust2 in the source material, so reproducibility of the exact plot styling is not version-pinned
- The README notes that when the main pipeline is inconvenient, individual functions must be run stepwise (ko2kegg_abundance, pathway_daa, pathway_annotation, visualization), which adds manual bookkeeping

## Evidence

- [abstract] abstract quote: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals."
- [intro] intro quote: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] readme quote: [evidence span withheld — anchor status: external_doc]
