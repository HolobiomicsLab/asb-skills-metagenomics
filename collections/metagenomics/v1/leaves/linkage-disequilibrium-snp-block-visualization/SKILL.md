---
name: linkage-disequilibrium-snp-block-visualization
description: Use when you have mbQTL taxa-SNP rho correlation results and need to
  visualize SNVs in linkage disequilibrium (e.g., a chr15 LD block) as correlated
  clusters in a heatmap.
when_to_use_negative:
- Taxa are presence/absence (categorical) and require logistic regression via mbQTL's
  logistic regression analysis rather than rho correlation
- The goal is significant taxa-SNP relationship testing via linear regression (MatrixEQTL-based
  mbQTL Part A), not LD visualization
- Input is a raw phyloseq or metagenomeSeq object that has not yet been converted/normalized
  via phyloseq_to_metagenomeSeq() or metagenomeSeqToMbqtl
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mbQTL
  role: Provides taxaSnpCor(), allToAllProduct(), and mbQtlCorHeatmap() for rho estimation
    and LD block heatmap rendering
  repo: https://github.com/Mercedeh66/mbQTL
- name: R
  role: Runtime environment in which mbQTL functions are called to produce the rho
    correlation heatmap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:735aef73c75461ce63af9add7f4cb29bf90d3d33ef02544accb2f8aa370203fe
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
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# linkage-disequilibrium-snp-block-visualization

## Summary

Estimate rho correlations between all SNPs and all taxa with mbQTL's taxaSnpCor() applied to allToAllProduct() output and taxa-taxa correlation matrices, then render the rho table as a taxa-by-SNP heatmap with mbQtlCorHeatmap() to expose SNVs in linkage disequilibrium as visibly correlated blocks.

## When to use

Use when you have mbQTL taxa-SNP rho correlation results and need to visualize SNVs in linkage disequilibrium (e.g., a chr15 LD block) as correlated clusters in a heatmap.

## When NOT to use

- Taxa are presence/absence (categorical) and require logistic regression via mbQTL's logistic regression analysis rather than rho correlation
- The goal is significant taxa-SNP relationship testing via linear regression (MatrixEQTL-based mbQTL Part A), not LD visualization
- Input is a raw phyloseq or metagenomeSeq object that has not yet been converted/normalized via phyloseq_to_metagenomeSeq() or metagenomeSeqToMbqtl

## Inputs

- trimmed taxa-SNP rho correlation table (taxa_SNP_Cor_lim) from mbQTL's taxaSnpCor() step
- allToAllProduct() output of SNP rsID correlations (for_all_rsids)
- taxa-taxa correlation matrix (correlationMicrobes)
- mbQTL R package environment

## Outputs

- taxa_SNP_Cor: taxa-by-SNP rho correlation table
- rho_cor_heatmap.png: rendered heatmap showing LD SNV blocks (e.g., chr15) as correlated clusters

## How to apply

In R, load the mbQTL package and the trimmed taxa_SNP_Cor_lim rho correlation table produced by the upstream correlation/trimming step. The rho values are estimated by applying taxaSnpCor() to the allToAllProduct() output (for_all_rsids) together with the taxa-taxa correlations (correlationMicrobes), which adjusts taxa-SNP associations after estimating taxa-taxa R values — this is what allows simultaneous estimation of taxa-SNP associations and identification of LD SNPs and regions. Call mbQtlCorHeatmap() on the trimmed rho table to render the taxa-by-SNP rho estimates as a heatmap, then save the figure as rho_cor_heatmap.png. Inspect the heatmap so that SNVs in linkage disequilibrium appear as visibly correlated blocks — in the example dataset this is the chr15 block — matching Fig. 1 of the paper.

## Related tools

- **mbQTL** (Provides taxaSnpCor(), allToAllProduct(), and mbQtlCorHeatmap() for rho estimation and LD block heatmap rendering) — https://github.com/Mercedeh66/mbQTL
- **R** (Runtime environment in which mbQTL functions are called to produce the rho correlation heatmap)

## Examples

```
library(mbQTL); p <- mbQtlCorHeatmap(taxa_SNP_Cor_lim); png('rho_cor_heatmap.png'); print(p); dev.off()
```

## Evaluation signals

- rho_cor_heatmap.png file exists after rendering
- The chr15 LD SNV block is visually identifiable as a correlated cluster in the heatmap
- Heatmap layout is taxa-by-SNP with rho values from the trimmed taxa_SNP_Cor_lim table
- Result is consistent with the Fig. 1 presentation of the mbQTL paper

## Limitations

- The rho estimation is a novel method contingent on first estimating taxa-taxa R values before adjusting for taxa-SNP association; the upstream correlation/trimming step must be run first
- The vignette datasets (e.g., Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt) are simulated and not factual, so block appearance may differ on real data

## Evidence

- [intro] Estimate rho values for taxa-SNP correlation after estimating taxa-taxa R values, allowing simultaneous estimation of taxa-SNP associations and identification of linkage disequilibrium (LD) SNPs and regions: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [other] Workflow renders the trimmed rho table as a heatmap with mbQtlCorHeatmap to expose LD blocks: [evidence span withheld — anchor status: not_found]
- [other] Heatmap should match Fig. 1 of the paper with the chr15 LD block visible: [evidence span withheld — anchor status: not_found]
- [readme] Installation via Bioconductor: [evidence span withheld — anchor status: external_doc]
