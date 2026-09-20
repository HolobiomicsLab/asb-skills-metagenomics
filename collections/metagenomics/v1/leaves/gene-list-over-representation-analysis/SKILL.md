---
name: gene-list-over-representation-analysis
description: Use when you have a list of differentially expressed genes (e.g. DESeq2
  output with gene name/id, log2 fold change, and p-adjusted values) and want to test
  whether GO terms or KEGG pathways are over-represented, with multiple-testing correction.
when_to_use_negative:
- The goal is gene set enrichment analysis (GSEA) on a full ranked gene list rather
  than over-representation of a selected gene list — use ClusterProfShinyGSEA instead
- The input is single-cell RNA-seq data requiring Seurat-based analysis — use SeuratV3Wizard,
  not the enrichment apps
- The input is metagenomic (e.g. 16S) data requiring differential metagenomic analysis
  — use SHAMAN, not the enrichment apps
edam_operation: http://edamontology.org/operation_2434
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: ClusterProfShinyORA
  role: Web-based Shiny app in NASQAR that wraps clusterProfiler to run over-representation
    analysis of a gene list against GO terms and KEGG pathways
  repo: https://github.com/nasqar/ClusterProfShinyORA
- name: clusterProfiler
  role: Underlying R package implementing over-representation analysis and functional
    profile visualization
  repo: https://github.com/GuangchuangYu/clusterProfiler
- name: Pathview
  role: R package used to render enriched KEGG pathway plots for significant pathways
- name: DESeq2Shiny
  role: Upstream app providing analyzed DESeq2 differential expression data that can
    be imported directly into ClusterProfShinyORA
  repo: https://github.com/nasqar/deseq2shiny
- name: NASQAR
  role: Web-based platform hosting the ClusterProfShinyORA enrichment app
  repo: https://github.com/nasqar/NASQAR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03577-4
    title: 'NASQAR: a web-based platform for high-throughput sequencing data analysis
      and visualization'
  source_content_hash: sha256:116e43de56cec901cc2bbc90bc7cb471fd20f363ad21f86f2bce6cf4c6238485
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
  edam_operation: http://edamontology.org/operation_2434
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gene-list-over-representation-analysis

## Summary

Performs over-representation analysis (ORA) of a gene list against Gene Ontology (GO) terms and KEGG pathways using the clusterProfiler-backed ClusterProfShinyORA app in NASQAR, producing enrichment tables with adjusted p-values and Pathview KEGG pathway plots. It wraps a GUI around clusterProfiler so non-programmers can run functional enrichment without writing R.

## When to use

Use when you have a list of differentially expressed genes (e.g. DESeq2 output with gene name/id, log2 fold change, and p-adjusted values) and want to test whether GO terms or KEGG pathways are over-represented, with multiple-testing correction.

## When NOT to use

- The goal is gene set enrichment analysis (GSEA) on a full ranked gene list rather than over-representation of a selected gene list — use ClusterProfShinyGSEA instead
- The input is single-cell RNA-seq data requiring Seurat-based analysis — use SeuratV3Wizard, not the enrichment apps
- The input is metagenomic (e.g. 16S) data requiring differential metagenomic analysis — use SHAMAN, not the enrichment apps

## Inputs

- Gene list (e.g. Supplementary Additional file 1 example list)
- Differential gene expression table (comma or tab delimited .csv/.txt, e.g. DESeq2 output with required columns: Gene name/id, Log2 Fold Change, p-adjusted values)
- Data imported from the upstream DESeq2Shiny app

## Outputs

- ORA results table with enriched GO terms, enrichment statistics, and adjusted p-values
- ORA results table with enriched KEGG pathways, enrichment statistics, and adjusted p-values
- Enriched KEGG pathway plots generated with Pathview

## How to apply

Load a gene list (e.g. the example list from the article's Supplementary Additional file 1) or a DGE table into ClusterProfShinyORA, either by uploading a DESeq2 output file or importing analyzed data from the upstream DESeq2Shiny app. Run over-representation analysis against the GO and KEGG annotation databases, backed by the clusterProfiler R package, with multiple-testing (p-adjusted) correction. Inspect the resulting EnrichGo/EnrichKEGG results tables for enriched terms and their enrichment statistics and adjusted p-values. For significant KEGG pathways, generate pathway plots with Pathview. Save the ORA results table and pathway plots as output files; correctness is judged by the presence of enriched GO/KEGG terms with enrichment statistics and adjusted p-values and rendered pathway plots for significant pathways.

## Related tools

- **ClusterProfShinyORA** (Web-based Shiny app in NASQAR that wraps clusterProfiler to run over-representation analysis of a gene list against GO terms and KEGG pathways) — https://github.com/nasqar/ClusterProfShinyORA
- **clusterProfiler** (Underlying R package implementing over-representation analysis and functional profile visualization) — https://github.com/GuangchuangYu/clusterProfiler
- **Pathview** (R package used to render enriched KEGG pathway plots for significant pathways)
- **DESeq2Shiny** (Upstream app providing analyzed DESeq2 differential expression data that can be imported directly into ClusterProfShinyORA) — https://github.com/nasqar/deseq2shiny
- **NASQAR** (Web-based platform hosting the ClusterProfShinyORA enrichment app) — https://github.com/nasqar/NASQAR

## Examples

```
docker run -p 80:80 aymanm/nasqarall
```

## Evaluation signals

- An ORA results table is produced containing enriched GO/KEGG terms with enrichment statistics and adjusted p-values
- The input file satisfies the schema requirement: comma- or tab-delimited table with Gene name/id, Log2 Fold Change, and p-adjusted columns selected
- Significant KEGG pathways are rendered as pathway plots via Pathview
- The results table is downloadable/displayable (EnrichGo and EnrichKEGG results tables present)
- Multiple-testing correction is applied (adjusted p-values reported rather than raw p-values)

## Limitations

- GeneCountMerger-style preprocessing caveat applies to gene-list data: if gene columns/identifiers do not match across files, the workflow will not work (first column must contain the genes)
- Users must manually map uploaded file columns to the required fields (Gene name/id, Log2 Fold Change, p-adjusted values)
- Supplementary example data (Additional file 1) is referenced but its content is not included in the article text, so reproducing the example use case requires obtaining that file separately
- Over-representation requires a pre-selected gene list; the full ranked-list GSEA mode is a separate app (ClusterProfShinyGSEA)

## Evidence

- [full_text] The ClusterProfilerShiny apps allow quick and easy over-representation analysis (ORA) and gene set enrichment analysis (GSEA) of GO terms and KEGG pathways: [evidence span withheld — anchor status: not_found]
- [results] Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app.: "Users can upload their own data from the output of DESeq2, for example, or import analyzed data from the upstream DESeq2Shiny app."
- [readme] This app allows for quick and easy **Gene Set Enrichment Analysis (GSEA)** of GO-Terms and KEGG pathways.: [evidence span withheld — anchor status: external_doc]
- [readme] The required columns are **Gene name/id, Log2 Fold Change, p-adjusted values**: [evidence span withheld — anchor status: external_doc]
- [methods] analysis of differential mRNA expression, metagenomics analysis, and functional enrichment analysis: "analysis of differential mRNA expression, metagenomics analysis, and functional enrichment analysis"
