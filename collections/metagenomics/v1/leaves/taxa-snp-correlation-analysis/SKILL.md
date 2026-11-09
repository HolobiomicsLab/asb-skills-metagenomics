---
name: taxa-snp-correlation-analysis
description: Use when estimating rho correlations between SNPs and microbial taxa
  across a sample cohort while simultaneously controlling for taxa–taxa correlation
  (R) and goodness-of-fit (R²) to flag LD SNPs/regions, per mbQTL Approach B.
when_to_use_negative:
- The goal is linear regression testing of taxa–SNP relationships, which mbQTL handles
  separately via its core MatrixEQTL model (Approach A), not the rho-estimation correlation
  workflow.
- The taxa are presence/absence (categorical), where the article prescribes logistic
  regression to estimate the relationship of one SNP and one taxa instead.
- Input is a phyloseq S4 object or metagenomeSeq MRexperiment object not yet converted
  — normalize/aggregate and convert via phyloseq_to_metagenomeSeq() and metagenomeSeqToMbqtl
  first.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: mbQTL
  role: R/Bioconductor package providing coringTaxa, allToAllProduct, and taxaSnpCor
    used for the Approach B rho-estimation workflow
  repo: github.com/Mercedeh66/mbQTL
- name: R
  role: Runtime environment in which the mbQTL functions are called on the SnpFile
    and microbeAbund vignette datasets
- name: MatrixEQTL
  role: Core linear regression engine underlying mbQTL's complementary regression
    approach (Approach A); not used in the taxaSnpCor rho step itself
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:adf608a082288fbbbf046876667f7e4a05b3f15c5bc2fcc28e304389cb156113
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# taxa-snp-correlation-analysis

## Summary

Reproduces mbQTL's Approach B correlation workflow: estimate taxa–taxa R values with coringTaxa, retain extreme taxa–taxa correlations with allToAllProduct(probs=c(0.0001,0.9999)), then call taxaSnpCor to estimate rho for all SNP–taxa pairs adjusted for those R values.

## When to use

Use when estimating rho correlations between SNPs and microbial taxa across a sample cohort while simultaneously controlling for taxa–taxa correlation (R) and goodness-of-fit (R²) to flag LD SNPs/regions, per mbQTL Approach B.

## When NOT to use

- The goal is linear regression testing of taxa–SNP relationships, which mbQTL handles separately via its core MatrixEQTL model (Approach A), not the rho-estimation correlation workflow.
- The taxa are presence/absence (categorical), where the article prescribes logistic regression to estimate the relationship of one SNP and one taxa instead.
- Input is a phyloseq S4 object or metagenomeSeq MRexperiment object not yet converted — normalize/aggregate and convert via phyloseq_to_metagenomeSeq() and metagenomeSeqToMbqtl first.

## Inputs

- microbeAbund: taxa-by-sample microbial abundance data frame (rownames taxa, colnames sample names)
- SnpFile: SNP-by-sample genotype data frame (rownames snps, colnames sample names)

## Outputs

- correlationMicrobes object: taxa–taxa correlation (R) matrix from coringTaxa
- for_all_rsids object: allToAllProduct output with SNP–taxa R² for all SNPs
- rho_estimates.tsv: table of rho estimates with taxa, SNP, and rho columns

## How to apply

Load mbQTL in R and attach the simulated vignette datasets with data("SnpFile") and data("microbeAbund"). Call coringTaxa(microbeAbund) to compute the taxa–taxa correlation matrix (R values) and goodness-of-fit R² measures. Run allToAllProduct on the coringTaxa output with probs = c(0.0001, 0.9999) to retain only the most extreme taxa–taxa correlations. Then pass the coringTaxa output and the allToAllProduct output (for_all_rsids) to taxaSnpCor, which estimates rho values for all SNP–taxa pairs adjusted for the taxa–taxa R values. Jointly evaluate rho and R² measures to identify taxa–SNP associations and flag potential linkage disequilibrium SNPs/regions, then write results to rho_estimates.tsv.

## Related tools

- **mbQTL** (R/Bioconductor package providing coringTaxa, allToAllProduct, and taxaSnpCor used for the Approach B rho-estimation workflow) — github.com/Mercedeh66/mbQTL
- **R** (Runtime environment in which the mbQTL functions are called on the SnpFile and microbeAbund vignette datasets)
- **MatrixEQTL** (Core linear regression engine underlying mbQTL's complementary regression approach (Approach A); not used in the taxaSnpCor rho step itself)

## Examples

```
correlationMicrobes <- coringTaxa(microbeAbund); for_all_rsids <- allToAllProduct(correlationMicrobes, probs=c(0.0001,0.9999)); rho <- taxaSnpCor(correlationMicrobes, for_all_rsids)
```

## Evaluation signals

- rho_estimates.tsv exists and is a well-formed table with taxa, SNP, and rho estimate columns.
- coringTaxa output is a square taxa-by-taxa table with off-diagonal correlation values within [-1, 1] and one R² goodness-of-fit entry per taxon pair.
- The rho values reproduce the taxa–SNP correlations adjusted for taxa–taxa R values as described for mbQTL's Approach B.
- The allToAllProduct call uses probs = c(0.0001, 0.9999) so only the most extreme taxa–taxa correlations are retained before taxaSnpCor.
- Single-rsID (allToAllProduct(SnpFile, microbeAbund, "chr1.171282963_T")) and all-SNP R² outputs match the vignette tables.

## Limitations

- The vignette datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt, covariates.txt) are simulated and are not factual, so results validate structure/behavior rather than real biology.
- Taxa–taxa correlations must be estimated (R values) before rho estimation; taxaSnpCor cannot be applied without the coringTaxa output.
- Only extreme taxa–taxa correlations are retained in the allToAllProduct quantile step (probs = c(0.0001, 0.9999)), so weaker taxa–taxa correlations are excluded from the adjustment.

## Evidence

- [intro] rho estimation after R value estimation: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] simultaneous evaluation of rho and R²: "we find clusters of taxa associated with various SNPs across the dataset by evaluating and controlling for both these measures together"
- [readme] Bioconductor install path: [evidence span withheld — anchor status: external_doc]
