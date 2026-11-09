---
name: gene-set-enrichment-analysis
description: Use when a researcher has a differential gene expression table (e.g.
  DESeq2 output with gene id, log2 fold change, adjusted p-value) and needs to identify
  enriched GO terms or KEGG pathways without writing R code.
when_to_use_negative:
- Input is raw RNA-seq count data that has not yet undergone differential expression
  analysis — route it through DESeq2Shiny or DESeq2 first
- The goal is merging or preprocessing individual gene count files — use GeneCountMerger
  instead
- The organism is not covered by the GO/KEGG annotation databases the clusterProfiler
  apps query
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: ClusterProfShinyORA
  role: Web-based over-representation analysis (ORA) app on GO terms and KEGG pathways,
    backed by clusterProfiler
  repo: https://github.com/nasqar/ClusterProfShinyORA
- name: ClusterProfShinyGSEA
  role: Web-based gene set enrichment analysis (GSEA) app on GO terms and KEGG pathways,
    backed by clusterProfiler
  repo: https://github.com/nasqar/ClusterProfShinyGSEA
- name: clusterProfiler
  role: R package implementing the enrichment methods and visualizations wrapped by
    the Shiny apps
  repo: https://github.com/GuangchuangYu/clusterProfiler
- name: Pathview
  role: R package used to render enriched KEGG pathway plots for significant pathways
- name: DESeq2Shiny
  role: Upstream app from which analyzed differential expression data can be imported
    into the enrichment apps
  repo: https://github.com/nasqar/deseq2shiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:5e69acad439fc54fc2d079c03b08746996b0bae924937dda4bf87687d5fa57ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nasqar/ClusterProfShinyORA
  tools:
  - ClusterProfShinyGSEA
  - ClusterProfShinyORA
  - DESeq2Shiny
  - NASQAR
  - Pathview
  - R
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-set-enrichment-analysis

## Summary

Performs over-representation analysis (ORA) and gene set enrichment analysis (GSEA) of GO terms and KEGG pathways on RNA-seq differential expression data, as implemented by the NASQAR ClusterProfilerShiny apps wrapping the clusterProfiler R package.

## When to use

Use when a researcher has a differential gene expression table (e.g. DESeq2 output with gene id, log2 fold change, adjusted p-value) and needs to identify enriched GO terms or KEGG pathways without writing R code.

## When NOT to use

- Input is raw RNA-seq count data that has not yet undergone differential expression analysis — route it through DESeq2Shiny or DESeq2 first
- The goal is merging or preprocessing individual gene count files — use GeneCountMerger instead
- The organism is not covered by the GO/KEGG annotation databases the clusterProfiler apps query

## Inputs

- Differential gene expression (DGE) table, e.g. DESeq2 output (.csv/.txt, comma- or tab-delimited) with required columns: Gene name/id, Log2 Fold Change, p-adjusted values
- Gene list loaded from Supplementary Additional file 1
- Data imported from the upstream DESeq2Shiny app

## Outputs

- EnrichGo results table (enriched GO terms with enrichment statistics and adjusted p-values)
- EnrichKEGG results table (enriched KEGG pathways with multiple-testing-corrected statistics)
- Visualizations: bar plots, dot plots, enrichment plot maps, GO induced graphs (goplot, cnetplot), GSEA plots
- Enriched KEGG pathway plots generated with Pathview

## How to apply

Load a gene list or DESeq2 differential expression table (comma- or tab-delimited, with Gene name/id, Log2 Fold Change, and p-adjusted values columns selected by the user) into ClusterProfShinyORA or ClusterProfShinyGSEA, either by direct upload or by importing analyzed data from the upstream DESeq2Shiny app. For ClusterProfShinyORA, run over-representation analysis (enrichGO/enrichKEGG) against GO and KEGG annotation databases with multiple-testing correction on the submitted gene list. For ClusterProfShinyGSEA, run gene set enrichment analysis on the ranked gene list. Visualize results with bar plots, dot plots, enrichment map plots, GO induced graphs (goplot/cnetplot), and render enriched KEGG pathway plots with the Pathview package. Export the EnrichGo and EnrichKEGG results tables and pathway plots as output files.

## Related tools

- **ClusterProfShinyORA** (Web-based over-representation analysis (ORA) app on GO terms and KEGG pathways, backed by clusterProfiler) — https://github.com/nasqar/ClusterProfShinyORA
- **ClusterProfShinyGSEA** (Web-based gene set enrichment analysis (GSEA) app on GO terms and KEGG pathways, backed by clusterProfiler) — https://github.com/nasqar/ClusterProfShinyGSEA
- **clusterProfiler** (R package implementing the enrichment methods and visualizations wrapped by the Shiny apps) — https://github.com/GuangchuangYu/clusterProfiler
- **Pathview** (R package used to render enriched KEGG pathway plots for significant pathways)
- **DESeq2Shiny** (Upstream app from which analyzed differential expression data can be imported into the enrichment apps) — https://github.com/nasqar/deseq2shiny

## Evaluation signals

- ORA results table is produced containing enriched GO/KEGG terms with enrichment statistics and adjusted p-values
- Selected input columns match the required schema: gene name/id, log2 fold change, and adjusted p-value
- Significant KEGG pathways are successfully rendered as Pathview pathway plots
- EnrichGo and EnrichKEGG results tables are downloadable/displayed without parse errors
- ORA results are reproducible against the tutorial workflow using the example (demo) Drosophila DE data

## Limitations

- Multiple-testing correction applies, but the apps require a gene identifier type the GO/KEGG databases recognize; unmapped gene IDs will drop out
- For a sample gene-counts file, columns must be explicitly matched to Gene name/id, Log2 Fold Change, and p-adjusted values
- Enrichment is limited to GO terms and KEGG pathways as implemented in clusterProfiler; other gene set sources are not covered by these apps
- Live analysis depends on the NASQAR platform (public instance at nasqar.abudhabi.nyu.edu or a self-deployed Docker image)

## Evidence

- [results] Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app.: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
- [readme] This app allows for quick and easy **Gene Set Enrichment Analysis (GSEA)** of GO-Terms and KEGG pathways.: [evidence span withheld — anchor status: external_doc]
- [readme] The required columns are Gene name/id, Log2 Fold Change, p-adjusted values: [evidence span withheld — anchor status: external_doc]
- [supplementary] Generate enriched KEGG pathway plots for significant pathways using Pathview.: [evidence span withheld — anchor status: not_found]
