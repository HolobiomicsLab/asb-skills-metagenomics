---
name: kegg-pathway-visualization
description: Use when a DESeq2 differential expression table or ranked gene list has
  produced significant KEGG pathways via ClusterProfShinyORA or ClusterProfShinyGSEA
  and the next step is to visualize those enriched pathways as KEGG pathway plots.
when_to_use_negative:
- Input is raw gene count files rather than a DE results table with Gene name/id,
  Log2 Fold Change, and p-adjusted columns — preprocess first (e.g. GeneCountMerger,
  then DESeq2Shiny) before enrichment visualization.
- Enrichment targets only Gene Ontology terms with no KEGG pathway analysis — Pathview
  KEGG plots apply to KEGG pathways, while GO visuals (goplot, cnetplot, induced graphs)
  are the appropriate outputs.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: ClusterProfShinyGSEA
  role: Shiny app wrapping clusterProfiler to run GSEA of GO terms and KEGG pathways;
    its significant KEGG hits feed the Pathview plots
  repo: https://github.com/nasqar/ClusterProfShinyGSEA
- name: ClusterProfShinyORA
  role: Shiny app performing over-representation analysis against GO and KEGG annotation
    databases with multiple-testing correction to identify significant KEGG pathways
  repo: https://github.com/nasqar/ClusterProfShinyORA
- name: Pathview
  role: R package used to generate the enriched KEGG pathway plots for significant
    pathways
  repo: https://bioconductor.org/packages/release/bioc/html/pathview.html
- name: clusterProfiler
  role: Backend R package implementing ORA/GSEA functional profile analysis and EnrichKEGG/EnrichGo
    result tables
  repo: https://bioconductor.org/packages/release/bioc/html/clusterProfiler.html
- name: DESeq2Shiny
  role: Upstream differential expression app from which analyzed DESeq2 data can be
    imported directly into the enrichment apps
  repo: https://github.com/nasqar/deseq2shiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:ea54fc760840e6b5d3fab655f6edb83d467cd33a771e636fcde6104e31fe06ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/ClusterProfShinyGSEA
  tools:
  - ClusterProfShinyGSEA
  - ClusterProfShinyORA
  - DESeq2Shiny
  - NASQAR
  - Pathview
  - R
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kegg-pathway-visualization

## Summary

Render enriched KEGG pathway plots for significant KEGG pathways identified by NASQAR's ClusterProfilerShiny ORA/GSEA apps, which wrap the clusterProfiler R package, using the Pathview package to overlay DESeq2-derived genes and log2 fold changes onto KEGG pathway maps.

## When to use

Use when a DESeq2 differential expression table or ranked gene list has produced significant KEGG pathways via ClusterProfShinyORA or ClusterProfShinyGSEA and the next step is to visualize those enriched pathways as KEGG pathway plots.

## When NOT to use

- Input is raw gene count files rather than a DE results table with Gene name/id, Log2 Fold Change, and p-adjusted columns — preprocess first (e.g. GeneCountMerger, then DESeq2Shiny) before enrichment visualization.
- Enrichment targets only Gene Ontology terms with no KEGG pathway analysis — Pathview KEGG plots apply to KEGG pathways, while GO visuals (goplot, cnetplot, induced graphs) are the appropriate outputs.

## Inputs

- Differential gene expression (DGE) table (.csv/.txt, comma- or tab-delimited) from DESeq2 with required columns: Gene name/id, Log2 Fold Change, p-adjusted values
- Analyzed differential expression data imported from the upstream DESeq2Shiny app
- Example demo data (e.g. www/exampleData/drosphila_example_de.csv) bundled with ClusterProfShinyGSEA
- Example gene list from the article's Supplementary Additional file 1 (NASQAR enrichment workflow)

## Outputs

- EnrichKEGG results table (significant KEGG pathways with enrichment statistics and adjusted p-values)
- EnrichGo results table
- Enriched KEGG pathway plots rendered with the Pathview package
- Downloadable visualization files (bar plots, dot plots, enrichment map plots, gsea plots)

## How to apply

Upload a differential gene expression (DGE) table (comma- or tab-delimited .csv/.txt, e.g. DESeq2 output) whose required columns are Gene name/id, Log2 Fold Change, and p-adjusted values, or import analyzed data from the upstream DESeq2Shiny app; a bundled example DGE file (drosphila_example_de.csv) mirrors the Gene Set Enrichment tutorial for familiarization. Run ClusterProfShinyORA (over-representation analysis) or ClusterProfShinyGSEA (gene set enrichment analysis), both backed by clusterProfiler, against GO and KEGG annotation databases with multiple-testing correction. From the EnrichKEGG results table, take the significant KEGG pathways and render them as Pathview plots; note that Pathview plots are KEGG-specific while GO enrichment uses other visuals (goplot, cnetplot, dot plots, enrichment maps). Decision point: GSEA takes a full ranked DGE list, whereas ORA operates on a gene list derived from significant DE genes. Finally, save the ORA results table and the KEGG pathway plot as output files, matching the article's example use case validation.

## Related tools

- **ClusterProfShinyGSEA** (Shiny app wrapping clusterProfiler to run GSEA of GO terms and KEGG pathways; its significant KEGG hits feed the Pathview plots) — https://github.com/nasqar/ClusterProfShinyGSEA
- **ClusterProfShinyORA** (Shiny app performing over-representation analysis against GO and KEGG annotation databases with multiple-testing correction to identify significant KEGG pathways) — https://github.com/nasqar/ClusterProfShinyORA
- **Pathview** (R package used to generate the enriched KEGG pathway plots for significant pathways) — https://bioconductor.org/packages/release/bioc/html/pathview.html
- **clusterProfiler** (Backend R package implementing ORA/GSEA functional profile analysis and EnrichKEGG/EnrichGo result tables) — https://bioconductor.org/packages/release/bioc/html/clusterProfiler.html
- **DESeq2Shiny** (Upstream differential expression app from which analyzed DESeq2 data can be imported directly into the enrichment apps) — https://github.com/nasqar/deseq2shiny

## Evaluation signals

- The EnrichKEGG results table is produced containing KEGG pathways with enrichment statistics and multiple-testing-adjusted p-values.
- Pathview plots are rendered only for the significant KEGG pathways returned by the ORA/GSEA run, not for all tested pathways.
- The uploaded DGE file's columns were successfully matched to the required Gene name/id, Log2 Fold Change, and p-adjusted values columns in the app.
- The ORA results table and the KEGG pathway plot are both saved as downloadable output files, as in the article's example use case.
- Running the bundled example data (drosphila_example_de.csv) reproduces the tutorial's expected enriched pathway outputs.

## Limitations

- Pathview plots are specific to KEGG pathways; GO term enrichment must be visualized with other plot types (goplot, cnetplot, dot plots, enrichment maps).
- Users must manually select which of their file's columns correspond to the required Gene name/id, Log2 Fold Change, and p-adjusted values; a missing or mislabeled column breaks the analysis.
- The KEGG pathway rendering depends on KEGG annotation database coverage via clusterProfiler; pathways absent from the annotation cannot be plotted.
- Instructions for launching NASQAR and the example use-case gene list live in Supplementary Additional file 1, which is not included in the main article text.

## Evidence

- [results] Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app.: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
- [readme] Visuals produced include dot plots, category net plots, enrichment map plots, GO induced graphs, gsea plots, and enriched KEGG pathway plots using the Pathview package.: [evidence span withheld — anchor status: not_found]
- [readme] A .csv/.txt file that contains a table of differential gene expression (DGE) data: [evidence span withheld — anchor status: not_found]
- [intro] By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently: "By decoupling data preprocessing, RNA-seq analysis, and gene enrichment applications from each other, users can leverage these functions independently"
