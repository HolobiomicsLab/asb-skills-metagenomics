---
name: taxa-taxa-correlation-product-analysis
description: Use when you have a SNP genotype matrix (SnpFile) and a microbial abundance
  matrix (microbeAbund, taxa rows x sample columns) and need SNP-taxa correlation
  (R2/rho) estimates plus taxa-taxa R values for mbQTL discovery.
when_to_use_negative:
- Input is a metagenomeSeq MRexperiment or phyloseq S4 object rather than a plain
  taxa/SNP x sample data frame — convert first via metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq()
- You need logistic regression on presence/absence (categorical) taxa, which allToAllProduct
  does not perform
- The data are not simulated matched SNP/taxa matrices with shared sample names (the
  vignette datasets are simulated, e.g. SNP_table_MVP_cor.txt / Taxa_table_MVP_cor.txt)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mbQTL
  role: R/Bioconductor package providing the allToAllProduct function that computes
    taxa-taxa R values and SNP-taxa R2/rho correlations
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment in which the mbQTL vignette workflow is executed (data(),
    allToAllProduct calls)
- name: MatrixEQTL
  role: Core linear regression model underlying mbQTL's SNP-taxa association estimation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:1ded64083582d4b233587a6b0f22d0a9b61128e5c9001da8746443991739463b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# taxa-taxa-correlation-product-analysis

## Summary

Use the mbQTL package's allToAllProduct function in R to compute taxa-taxa R values and the R2 (rho) correlation between host SNPs and 16S rRNA-derived taxa abundances, either for a single specified rsID or across all SNPs in the cohort.

## When to use

Use when you have a SNP genotype matrix (SnpFile) and a microbial abundance matrix (microbeAbund, taxa rows x sample columns) and need SNP-taxa correlation (R2/rho) estimates plus taxa-taxa R values for mbQTL discovery.

## When NOT to use

- Input is a metagenomeSeq MRexperiment or phyloseq S4 object rather than a plain taxa/SNP x sample data frame — convert first via metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq()
- You need logistic regression on presence/absence (categorical) taxa, which allToAllProduct does not perform
- The data are not simulated matched SNP/taxa matrices with shared sample names (the vignette datasets are simulated, e.g. SNP_table_MVP_cor.txt / Taxa_table_MVP_cor.txt)

## Inputs

- SnpFile: SNP genotype matrix with rownames as SNPs (e.g. chr1.171282963_T) and colnames as sample names
- microbeAbund: microbial abundance matrix with rownames as taxa and colnames as sample names
- optional single rsID string (e.g. "chr1.171282963_T") to restrict to one SNP
- optional CovFile: covariate matrix with rownames as covariates and colnames as sample names

## Outputs

- Table of R2 (rho) correlations between each SNP and all taxa across the sample cohort
- Taxa-taxa R-value correlation matrix
- Restricted per-SNP R2/rho table when a single rsID is supplied

## How to apply

Load mbQTL in R and attach the packaged example data with data("SnpFile") and data("microbeAbund"); input files must be matrices with rownames as taxa or SNPs and colnames as sample names. Run allToAllProduct(SnpFile, microbeAbund) to compute the all-to-all product: it first estimates taxa-taxa R values, then yields the correlation (R2/rho) between each SNP and all taxa across the sample cohort, following the article's rho estimation scheme that adjusts taxa-SNP association after taxa-taxa R estimation. To restrict the analysis to a single rsID (e.g. "chr1.171282963_T"), pass it as the third argument. mbQTL's linear regression core is based on the MatrixEQTL model. Record the resulting R2/rho tables for both calls and verify they match the single-rsID and all-SNP vignette outputs.

## Related tools

- **mbQTL** (R/Bioconductor package providing the allToAllProduct function that computes taxa-taxa R values and SNP-taxa R2/rho correlations) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment in which the mbQTL vignette workflow is executed (data(), allToAllProduct calls))
- **MatrixEQTL** (Core linear regression model underlying mbQTL's SNP-taxa association estimation)

## Examples

```
library(mbQTL); data("SnpFile"); data("microbeAbund"); res <- allToAllProduct(SnpFile, microbeAbund, "chr1.171282963_T")
```

## Evaluation signals

- The all-to-all call returns taxa-taxa R values plus an R2/rho table covering every SNP against every taxon, while the single-SNP call returns the restricted subset for that rsID only
- R2/rho values from the single-rsID call (e.g. "chr1.171282963_T") match the corresponding rows of the all-SNP call and the vignette outputs
- Input matrices have rownames (SNPs/taxa) and colnames (sample names) consistent with the SnpFile/microbeAbund schema
- Reproducing the workflow on the packaged data("SnpFile") and data("microbeAbund") yields the documented vignette tables

## Limitations

- The vignette datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt) are simulated and not factual
- rho estimation depends on first estimating taxa-taxa R values, then adjusting for taxa-SNP association; results are correlation-based, not effect-size estimates from logistic regression
- README states installation requires devtools::install_github('Mercedeh66/mbQTL') or BiocManager::install("mbQTL"); issues should be filed at the GitHub issues page

## Evidence

- [abstract] first R package integrating 16S rRNA with SNP data: "we introduce mbQTL, the first R package integrating 16S ribosomal RNA (rRNA) sequencing and single-nucleotide variation (SNV) and single-nucleotide polymorphism (SNP) data"
- [intro] rho estimation after taxa-taxa R values: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] vignette datasets are simulated: "The datasets used for this vignette are all simulated and are not factual."
- [readme] README installation and issue reporting: [evidence span withheld — anchor status: external_doc]
