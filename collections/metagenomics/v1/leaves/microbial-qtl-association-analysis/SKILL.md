---
name: microbial-qtl-association-analysis
description: Use when host genotype (SNP/SNV matrix) and 16S rRNA microbial abundance
  data are available and the goal is to find taxa-SNP associations and flag LD SNV
  blocks via rho estimation.
when_to_use_negative:
- The analysis goal is linear-regression mbQTL estimation via the core MatrixEQTL
  model (Approach A) rather than correlation/LD-aware rho estimation
- Taxa are presence/absence (categorical) and the aim is logistic regression of one
  SNP against one taxa
- Input is raw 16S reads rather than a normalized abundance table or metagenomeSeq
  object
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: mbQTL
  role: Provides coringTaxa, allToAllProduct, taxaSnpCor, and mbQtlCorHeatmap for
    rho estimation and LD visualization
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment for the mbQTL vignette workflow
- name: metagenomeSeq
  role: 'Optional upstream: normalize/aggregate an MRexperiment object via metagenomeSeqToMbqtl
    before analysis'
- name: phyloseq
  role: 'Optional upstream: convert phyloseq S4 objects to metagenomeSeq format via
    phyloseq_to_metagenomeSeq()'
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:56c968bebe1f8d6d30662fe16cf9270a4ef8ebe824a9fa4e96e37c518561cb24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  - metagenomeSeq
  - phyloseq
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# microbial-qtl-association-analysis

## Summary

Estimate rho correlations between SNPs/SNVs and microbial taxa with mbQTL's Approach B, which first estimates taxa-taxa R values and then adjusts taxa-SNP associations by them, jointly enabling identification of taxa-SNP associations and linkage disequilibrium (LD) SNP regions.

## When to use

Use when host genotype (SNP/SNV matrix) and 16S rRNA microbial abundance data are available and the goal is to find taxa-SNP associations and flag LD SNV blocks via rho estimation.

## When NOT to use

- The analysis goal is linear-regression mbQTL estimation via the core MatrixEQTL model (Approach A) rather than correlation/LD-aware rho estimation
- Taxa are presence/absence (categorical) and the aim is logistic regression of one SNP against one taxa
- Input is raw 16S reads rather than a normalized abundance table or metagenomeSeq object

## Inputs

- microbeAbund microbial abundance table (rownames taxa, colnames sample names)
- SnpFile SNP/SNV genotype table (rownames snps, colnames sample names)
- optional CovFile covariate table (rownames covariates, colnames sample names)
- metagenomeSeq MRexperiment object or phyloseq S4 object convertible via phyloseq_to_metagenomeSeq()

## Outputs

- taxa_SNP_Cor rho-estimate table with taxa, SNP, and rho columns
- trimmable/filtered rho table (e.g., taxa_SNP_Cor_lim) for heatmap rendering
- taxa-by-SNP rho heatmap (e.g., rho_cor_heatmap.png) showing LD SNP blocks

## How to apply

Load mbQTL in R and attach the vignette-style inputs: SnpFile (rownames SNPs, colnames samples) and microbeAbund (rownames taxa, colnames samples). Run coringTaxa() on microbeAbund to estimate taxa-taxa correlation (R) values across the sample cohort. Apply allToAllProduct() to the coringTaxa output with the extreme-quantile variant probs = c(0.0001, 0.9999) to retain only the most extreme taxa-taxa correlations. Pass the coringTaxa and allToAllProduct outputs to taxaSnpCor() to estimate rho values for all taxa-SNP pairs, adjusted for the taxa-taxa R values. Jointly evaluate rho and R-squared measures to identify taxa-SNP associations and flag potential LD SNPs/regions (e.g., the chr15 block in the example dataset), and optionally render the trimmed rho table with mbQtlCorHeatmap().

## Related tools

- **mbQTL** (Provides coringTaxa, allToAllProduct, taxaSnpCor, and mbQtlCorHeatmap for rho estimation and LD visualization) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment for the mbQTL vignette workflow)
- **metagenomeSeq** (Optional upstream: normalize/aggregate an MRexperiment object via metagenomeSeqToMbqtl before analysis)
- **phyloseq** (Optional upstream: convert phyloseq S4 objects to metagenomeSeq format via phyloseq_to_metagenomeSeq())

## Examples

```
data("SnpFile"); data("microbeAbund"); cor <- coringTaxa(microbeAbund); rsids <- allToAllProduct(cor, probs = c(0.0001, 0.9999)); rho <- taxaSnpCor(cor, rsids)
```

## Evaluation signals

- rho_estimates.tsv exists, is a well-formed table with taxa, SNP, and rho estimate columns
- rho values reproduce the taxa-SNP correlations adjusted for the previously estimated taxa-taxa R values as described for Approach B
- The heatmap (rho_cor_heatmap.png) shows LD SNVs (e.g., the chr15 block) as a visibly correlated cluster, consistent with Fig. 1 of the paper
- allToAllProduct used with probs = c(0.0001, 0.9999) so only the most extreme taxa-taxa correlations feed into taxaSnpCor

## Limitations

- The vignette datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt) are simulated and not factual, so absolute results are illustrative only
- Approach B requires jointly evaluating rho and R-squared measures; interpreting either alone risks mislabeling LD SNPs/regions as taxa-SNP associations
- No discussion section text is available in the provided article content

## Evidence

- [intro] rho estimation approach: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] LD identification rationale: "we find clusters of taxa associated with various SNPs across the dataset by evaluating and controlling for both these measures together"
- [abstract] package scope: "we introduce mbQTL, the first R package integrating 16S ribosomal RNA (rRNA) sequencing and single-nucleotide variation (SNV) and single-nucleotide polymorphism (SNP) data"
- [readme] installation: [evidence span withheld — anchor status: external_doc]
