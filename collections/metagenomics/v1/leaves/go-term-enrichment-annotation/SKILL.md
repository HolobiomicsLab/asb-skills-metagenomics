---
name: go-term-enrichment-annotation
description: Use when a differential expression result (e.g., a DESeq2 output table
  with gene name/id, log2 fold change, and padj) needs functional interpretation via
  enriched GO terms and KEGG pathways.
when_to_use_negative:
- Input is raw gene count files needing preprocessing or merging first — use GeneCountMerger
  or DESeq2Shiny upstream instead
- Input file lacks the required columns (Gene name/id, Log2 Fold Change, p-adjusted
  values) and column mapping cannot be done
- The analysis goal is differential expression itself rather than functional enrichment
  of an existing DE result
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3794
tools:
- name: ClusterProfShinyORA
  role: Web-based Shiny app performing over-representation analysis of GO terms and
    KEGG pathways backed by clusterProfiler
  repo: https://github.com/nasqar/ClusterProfShinyORA
- name: ClusterProfShinyGSEA
  role: Web-based Shiny app performing gene set enrichment analysis of GO terms and
    KEGG pathways backed by clusterProfiler
  repo: https://github.com/nasqar/ClusterProfShinyGSEA
- name: clusterProfiler
  role: Underlying R Bioconductor package implementing methods to analyze and visualize
    functional profiles of genes and gene clusters
  repo: https://github.com/GuangchuangYu/clusterProfiler
- name: Pathview
  role: R package used to render enriched KEGG pathway plots for significant pathways
  repo: https://bioconductor.org/packages/release/bioc/html/pathview.html
- name: DESeq2Shiny
  role: Upstream app from which analyzed differential expression data can be imported
    directly into the enrichment apps
  repo: https://github.com/nasqar/deseq2shiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:a2ac4d9aedc30b63d0b97664c06ef353929ba229abd4adcedf219869b04f90a6
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
  - http://edamontology.org/topic_3794
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# GO Term Enrichment Annotation

## Summary

Perform over-representation analysis (ORA) and gene set enrichment analysis (GSEA) of GO terms and KEGG pathways using the NASQAR ClusterProfilerShiny apps (ClusterProfShinyORA / ClusterProfShinyGSEA), which wrap the clusterProfiler R package for researchers without R programming experience.

## When to use

Use when a differential expression result (e.g., a DESeq2 output table with gene name/id, log2 fold change, and padj) needs functional interpretation via enriched GO terms and KEGG pathways.

## When NOT to use

- Input is raw gene count files needing preprocessing or merging first — use GeneCountMerger or DESeq2Shiny upstream instead
- Input file lacks the required columns (Gene name/id, Log2 Fold Change, p-adjusted values) and column mapping cannot be done
- The analysis goal is differential expression itself rather than functional enrichment of an existing DE result

## Inputs

- Differential gene expression table (.csv/.txt, comma- or tab-delimited; e.g. DESeq2 output) with required columns: Gene name/id, Log2 Fold Change, p-adjusted values
- Gene list for over-representation analysis (e.g. from Supplementary Additional file 1)
- Data imported from the upstream DESeq2Shiny app

## Outputs

- enrichGo results table
- enrichKEGG results table with enrichment statistics and adjusted p-values
- Dot plots, bar plots, enrichment map plots, GO induced graphs (goplot, cnetplot), GSEA plots
- Enriched KEGG pathway plots rendered via Pathview

## How to apply

Upload a differential gene expression table — either a comma- or tab-delimited DESeq2 output containing the required columns 'Gene name/id, Log2 Fold Change, p-adjusted values', or data imported directly from the upstream DESeq2Shiny app — into ClusterProfShinyORA (for ORA with a gene list) or ClusterProfShinyGSEA (for ranked-list GSEA). Run the analysis against Gene Ontology and KEGG annotation databases backed by the clusterProfiler package, with multiple-testing correction on the enrichment statistics. Generate visualizations: bar plots, dot plots, enrichment map plots, GO induced graphs (goplot/cnetplot for GO only), and enriched KEGG pathway plots rendered with the Pathview package. Download the enrichGo and enrichKEGG results tables and save significant pathway plots. Validation: results tables should contain enriched GO/KEGG terms with enrichment statistics and adjusted p-values, and significant KEGG pathways should render as pathway plots.

## Related tools

- **ClusterProfShinyORA** (Web-based Shiny app performing over-representation analysis of GO terms and KEGG pathways backed by clusterProfiler) — https://github.com/nasqar/ClusterProfShinyORA
- **ClusterProfShinyGSEA** (Web-based Shiny app performing gene set enrichment analysis of GO terms and KEGG pathways backed by clusterProfiler) — https://github.com/nasqar/ClusterProfShinyGSEA
- **clusterProfiler** (Underlying R Bioconductor package implementing methods to analyze and visualize functional profiles of genes and gene clusters) — https://github.com/GuangchuangYu/clusterProfiler
- **Pathview** (R package used to render enriched KEGG pathway plots for significant pathways) — https://bioconductor.org/packages/release/bioc/html/pathview.html
- **DESeq2Shiny** (Upstream app from which analyzed differential expression data can be imported directly into the enrichment apps) — https://github.com/nasqar/deseq2shiny

## Evaluation signals

- The enrichGo and enrichKEGG results tables are produced containing enriched terms with enrichment statistics and adjusted p-values
- Significant KEGG pathways render as Pathview pathway plots
- Uploaded DGE file maps to the three required columns (Gene name/id, Log2 Fold Change, p-adjusted values) before analysis proceeds
- Multiple-testing correction is applied so reported enrichment reflects adjusted p-values
- Visualization outputs (dot plot, bar plot, enrichment map, goplot/cnetplot) are generated and downloadable for the chosen GO/KEGG analysis

## Limitations

- The apps accept DESeq2-style output; other formats must be reshaped to the required columns before upload
- GO-specific visualizations (goplot, cnetplot induced graphs) apply to GO terms only, not KEGG
- ClusterProfShinyGSEA gene set enrichment runs 'where applicable' — ranked-list requirements must be met
- Example use-case details (Supplementary Additional file 1) are referenced but section content is not fully documented in the article text

## Evidence

- [readme] This app allows for quick and easy Gene Set Enrichment Analysis (GSEA) of GO-Terms and KEGG pathways.: [evidence span withheld — anchor status: external_doc]
- [results] Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app.: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
- [readme] The required columns are Gene name/id, Log2 Fold Change, p-adjusted values.: [evidence span withheld — anchor status: external_doc]
- [supplementary] Run ClusterProfShinyORA over-representation analysis backed by the clusterProfiler R package against Gene Ontology (GO) and KEGG annotation databases with multiple-testing correction.: [evidence span withheld — anchor status: not_found]
- [supplementary] The ORA results table is produced containing enriched GO/KEGG terms with enrichment statistics and adjusted p-values, and significant KEGG pathways are rendered as pathway plots.: [evidence span withheld — anchor status: not_found]
