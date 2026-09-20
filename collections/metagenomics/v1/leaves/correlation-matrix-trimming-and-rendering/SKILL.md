---
name: correlation-matrix-trimming-and-rendering
description: Use when you have a rho-value correlation table (taxa × SNP) produced
  by mbQTL and need to trim/visualize it as a heatmap to spot LD-linked SNV clusters.
when_to_use_negative:
- When the goal is significance testing of individual taxa-SNP pairs via linear regression
  (Part A uses the MatrixEQTL-based linear model, not correlation heatmaps).
- When taxa are categorical presence/absence and the relationship should be modeled
  by logistic regression instead of rho correlation.
- When input is a raw abundance/SNP table (microbeAbund/SnpFile) that has not yet
  been processed through mbQTL's rho-estimation workflow.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mbQTL
  role: R/Bioconductor package providing taxaSnpCor() for rho estimation and mbQtlCorHeatmap()
    for rendering the taxa-SNP correlation heatmap
  repo: https://github.com/Mercedeh66/mbQTL
- name: R
  role: Runtime environment in which mbQTL functions are called to trim and plot the
    rho table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:c7bc4c8e6485de2ca475e8e0a7e2eb4e98017f22c16a9810ac82628427d22faf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mercedeh66/mbQTL
  tools:
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# correlation-matrix-trimming-and-rendering

## Summary

A skill for rendering taxa-by-SNP rho correlation matrices as a heatmap using mbQTL's mbQtlCorHeatmap(), after trimming the rho table to focus on informative correlations (e.g., SNVs in linkage disequilibrium such as the chr15 block). It turns the taxa_SNP_Cor output of mbQTL's rho estimation into a Fig. 1-style visualization.

## When to use

Use when you have a rho-value correlation table (taxa × SNP) produced by mbQTL and need to trim/visualize it as a heatmap to spot LD-linked SNV clusters.

## When NOT to use

- When the goal is significance testing of individual taxa-SNP pairs via linear regression (Part A uses the MatrixEQTL-based linear model, not correlation heatmaps).
- When taxa are categorical presence/absence and the relationship should be modeled by logistic regression instead of rho correlation.
- When input is a raw abundance/SNP table (microbeAbund/SnpFile) that has not yet been processed through mbQTL's rho-estimation workflow.

## Inputs

- taxa_SNP_Cor rho correlation table (taxa × SNP rho values) from mbQTL taxaSnpCor() applied to allToAllProduct(for_all_rsids) output and correlationMicrobes
- trimmed rho table (taxa_SNP_Cor_lim) limiting rows/columns to taxa and SNVs of interest

## Outputs

- rho_cor_heatmap.png — rendered taxa-by-SNP rho correlation heatmap
- Visually identifiable correlated blocks representing SNVs in linkage disequilibrium (e.g., chr15 block), consistent with Fig. 1

## How to apply

First obtain the taxa_SNP_Cor rho table by applying taxaSnpCor() in mbQTL to the allToAllProduct() output (for_all_rsids) together with the taxa-taxa correlation matrix (correlationMicrobes). Trim the resulting rho table (e.g., taxa_SNP_Cor_lim) to the SNPs/taxa of interest before plotting, so LD blocks are not obscured by noise. Then call mbQtlCorHeatmap() on the trimmed rho table to render a taxa-by-SNP heatmap in which correlated blocks — such as SNVs in linkage disequilibrium on chr15 in the example dataset — appear as visible clusters, matching Fig. 1 of the mbQTL paper. Save the rendered figure (e.g., rho_cor_heatmap.png) and verify it exists and that the chr15 LD block is visually identifiable as a correlated cluster.

## Related tools

- **mbQTL** (R/Bioconductor package providing taxaSnpCor() for rho estimation and mbQtlCorHeatmap() for rendering the taxa-SNP correlation heatmap) — https://github.com/Mercedeh66/mbQTL
- **R** (Runtime environment in which mbQTL functions are called to trim and plot the rho table)

## Examples

```
mbQtlCorHeatmap(taxa_SNP_Cor_lim); ggsave('rho_cor_heatmap.png')
```

## Evaluation signals

- The output figure file (e.g., rho_cor_heatmap.png) exists on disk after execution.
- The heatmap shows SNVs in linkage disequilibrium (e.g., the chr15 block) as a visibly correlated cluster.
- The rendered figure matches the presentation of Fig. 1 of the mbQTL paper.
- The plotted matrix is a taxa-by-SNP rho table derived from taxa_SNP_Cor, not raw abundances or regression p-values.
- Trimming limits the heatmap to the SNPs/taxa of interest (taxa_SNP_Cor_lim) while preserving the LD block structure.

## Limitations

- The vignette datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt) are simulated and not factual, so visual patterns reflect the synthetic example rather than biological truth.
- mbQTL installation depends on BiocManager/devtools availability (devtools::install_github('Mercedeh66/mbQTL') or BiocManager::install('mbQTL')); failure modes around installation are handled via the GitHub issues tracker.
- Heatmap interpretation requires an upstream trimming step; rendering the full untrimmed taxa-SNP table may obscure LD blocks.

## Evidence

- [intro] rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated"
- [intro] allowing simultaneous estimation of taxa-SNP associations and identification of linkage disequilibrium (LD) SNPs and regions: [evidence span withheld — anchor status: not_found]
- [abstract] we introduce mbQTL, the first R package integrating 16S ribosomal RNA (rRNA) sequencing and single-nucleotide variation (SNV) and single-nucleotide polymorphism (SNP) data: "we introduce mbQTL, the first R package integrating 16S ribosomal RNA (rRNA) sequencing and single-nucleotide variation (SNV) and single-nucleotide polymorphism (SNP) data"
- [abstract] mbQTL is available on bioconductor at https://bioconductor.org/packages/mbQTL/.: "mbQTL is available on bioconductor at https://bioconductor.org/packages/mbQTL/."
- [readme] BiocManager::install("mbQTL"): [evidence span withheld — anchor status: external_doc]
