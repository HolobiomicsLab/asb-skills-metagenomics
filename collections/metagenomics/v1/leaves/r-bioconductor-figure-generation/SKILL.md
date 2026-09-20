---
name: r-bioconductor-figure-generation
description: Use when you have a trimmed taxa-SNP rho correlation table (e.g. taxa_SNP_Cor_lim)
  from mbQTL and need a heatmap figure to visually identify correlated SNV blocks
  such as LD regions.
when_to_use_negative:
- The rho correlation table has not yet been estimated/trimmed upstream — run taxaSnpCor()
  on allToAllProduct() and correlationMicrobes outputs first
- The goal is significance testing of taxa-SNP pairs via linear regression (MatrixeQTL
  model) rather than correlation-heatmap visualization
- Input is raw 16S abundance or genotype tables (microbeAbund/SnpFile) rather than
  a processed rho correlation result
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mbQTL
  role: R/Bioconductor package providing mbQtlCorHeatmap() to render the taxa-SNP
    rho heatmap; also supplies taxaSnpCor() for the upstream rho estimates
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment in which mbQTL figure generation is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:a5bab06883bfce54cbd6d9b77209c191d914e14bcb3750a58aa87854735e80d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# r-bioconductor-figure-generation

## Summary

Render taxa-by-SNP rho correlation estimates as a publication-quality heatmap in R using the mbQTL package's mbQtlCorHeatmap(), reproducing Figure 1 of the mbQTL paper and exposing linkage-disequilibrium (LD) SNV blocks such as the chr15 block.

## When to use

Use when you have a trimmed taxa-SNP rho correlation table (e.g. taxa_SNP_Cor_lim) from mbQTL and need a heatmap figure to visually identify correlated SNV blocks such as LD regions.

## When NOT to use

- The rho correlation table has not yet been estimated/trimmed upstream — run taxaSnpCor() on allToAllProduct() and correlationMicrobes outputs first
- The goal is significance testing of taxa-SNP pairs via linear regression (MatrixeQTL model) rather than correlation-heatmap visualization
- Input is raw 16S abundance or genotype tables (microbeAbund/SnpFile) rather than a processed rho correlation result

## Inputs

- taxa_SNP_Cor_lim: trimmed taxa-by-SNP rho correlation data frame produced by taxaSnpCor() applied to allToAllProduct() output and correlationMicrobes
- R session with mbQTL installed from Bioconductor or GitHub

## Outputs

- rho_cor_heatmap.png: rendered taxa-by-SNP rho heatmap figure (cf. Fig. 1)
- Visual identification of LD SNV blocks (e.g. chr15 correlated cluster)

## How to apply

Load the mbQTL R/Bioconductor package (installed via BiocManager::install("mbQTL") or devtools::install_github('Mercedeh66/mbQTL')) and read the trimmed taxa-SNP rho correlation result (taxa_SNP_Cor_lim) produced by the upstream taxaSnpCor()/trimming step. Call mbQtlCorHeatmap() on the trimmed rho table to render taxa-by-SNP rho estimates as a heatmap. Inspect the plot so that SNVs in linkage disequilibrium — e.g. the chr15 block in the example dataset — appear as visibly correlated clusters, matching Fig. 1 of the paper. Save the rendered figure as rho_cor_heatmap.png and verify the file exists and the chr15 LD block is identifiable as a correlated cluster. This visualization step builds on mbQTL's rho-estimation method, in which rho values are estimated after the taxa-taxa R value is computed and then adjusted for taxa-SNP association.

## Related tools

- **mbQTL** (R/Bioconductor package providing mbQtlCorHeatmap() to render the taxa-SNP rho heatmap; also supplies taxaSnpCor() for the upstream rho estimates) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment in which mbQTL figure generation is executed)

## Examples

```
library(mbQTL); mbQtlCorHeatmap(taxa_SNP_Cor_lim)
```

## Evaluation signals

- Figure file rho_cor_heatmap.png exists after rendering
- The chr15 LD SNV block is visually identifiable as a correlated cluster in the heatmap
- Heatmap axes correspond to taxa (rows) and SNPs (columns) of the trimmed taxa_SNP_Cor_lim table
- Rendering matches the Fig. 1 presentation of the mbQTL paper

## Limitations

- The paper's example datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt, covariates.txt) are simulated and not factual, so expected block patterns come from the vignette example rather than real data
- Heatmap interpretation depends on the quality of the upstream rho estimation; the rho method requires taxa-taxa R values to be estimated first and adjusted for taxa-SNP association

## Evidence

- [intro] rho estimation workflow: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [readme] installation: [evidence span withheld — anchor status: external_doc]
- [abstract] package scope: "We describe various statistical methods implemented for the identification of microbe-SNV pairs, relevant statistical measures, and plot functionality for interpretation."
