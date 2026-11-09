---
name: microbiome-genotype-association-analysis
description: Use when you have a SNP genotype matrix and a microbial taxa abundance
  matrix over the same sample cohort and need to identify significant taxa-SNP (mbQTL)
  relationships, including all-to-all R² estimates or a single specified rsID.
when_to_use_negative:
- Your taxa are presence/absence (categorical) and you need the one-SNP-one-taxa logistic
  regression workflow instead of the linear-regression allToAllProduct correlation
- Your microbiome data is not yet in taxa-by-sample data frame format (e.g. raw phyloseq/metagenomeSeq
  objects) and must first be normalized or converted with metagenomeSeqToMbqtl / phyloseq_to_metagenomeSeq()
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3305
tools:
- name: mbQTL
  role: R/Bioconductor package whose allToAllProduct function computes the R² correlation
    between SNPs and taxa; installed via devtools::install_github or BiocManager
  repo: github.com/Mercedeh66/mbQTL
- name: MatrixEQTL
  role: Core linear regression model underlying mbQTL's taxa-SNP association estimation
- name: R
  role: Runtime environment in which the mbQTL example datasets and functions are
    executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:832f43134688c186be18f0669e66d3b27fa43b09d33e5f3433df2dbdf1af0bf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# microbiome-genotype-association-analysis

## Summary

Estimate SNP–taxa associations (mbQTLs) with the mbQTL R/Bioconductor package, which computes R²/rho correlation values between host SNP genotypes and microbial taxa abundances using allToAllProduct and the core MatrixEQTL linear regression model.

## When to use

Use when you have a SNP genotype matrix and a microbial taxa abundance matrix over the same sample cohort and need to identify significant taxa-SNP (mbQTL) relationships, including all-to-all R² estimates or a single specified rsID.

## When NOT to use

- Your taxa are presence/absence (categorical) and you need the one-SNP-one-taxa logistic regression workflow instead of the linear-regression allToAllProduct correlation
- Your microbiome data is not yet in taxa-by-sample data frame format (e.g. raw phyloseq/metagenomeSeq objects) and must first be normalized or converted with metagenomeSeqToMbqtl / phyloseq_to_metagenomeSeq()

## Inputs

- SnpFile: SNP genotype data frame (rownames SNPs, colnames sample names)
- microbeAbund: microbial abundance data frame (rownames taxa, colnames sample names)
- optional CovFile: covariates data frame (rownames covariates, colnames sample names)
- optional single SNP/rsID string (e.g. "chr1.171282963_T")
- optional metagenomeSeq or phyloseq objects, converted first via metagenomeSeqToMbqtl / phyloseq_to_metagenomeSeq()

## Outputs

- Table of R² (rho) values correlating each SNP with all taxa across the sample cohort
- Taxa-taxa R value matrix from the all-to-all product
- Single-rsID R²/rho table when a specific SNP identifier is supplied

## How to apply

Load mbQTL in R and attach the packaged example datasets with data("SnpFile") and data("microbeAbund"), where SnpFile has rownames as SNPs and colnames as sample names and microbeAbund has rownames as taxa and colnames as sample names. Run allToAllProduct(SnpFile, microbeAbund) to compute the all-to-all product, yielding taxa-taxa R values and the R² correlation between each SNP and all taxa across the sample cohort. To restrict analysis to one rsID, pass the SNP identifier as a third argument, e.g. allToAllProduct(SnpFile, microbeAbund, "chr1.171282963_T"). The method estimates rho values for taxa-SNP correlation after estimating taxa-taxa R values, adjusted for association of taxa-SNP; the underlying linear regression uses the core MatrixEQTL model. If input is a metagenomeSeq MRexperiment or phyloseq object, first normalize via metagenomeSeqToMbqtl or convert with phyloseq_to_metagenomeSeq().

## Related tools

- **mbQTL** (R/Bioconductor package whose allToAllProduct function computes the R² correlation between SNPs and taxa; installed via devtools::install_github or BiocManager) — github.com/Mercedeh66/mbQTL
- **MatrixEQTL** (Core linear regression model underlying mbQTL's taxa-SNP association estimation)
- **R** (Runtime environment in which the mbQTL example datasets and functions are executed)

## Examples

```
data("SnpFile"); data("microbeAbund"); allToAllProduct(SnpFile, microbeAbund, "chr1.171282963_T")
```

## Evaluation signals

- Running allToAllProduct(SnpFile, microbeAbund) returns a table with the R² correlation of each SNP against all taxa, matching the all-SNP vignette output
- Running allToAllProduct(SnpFile, microbeAbund, "chr1.171282963_T") restricts results to one rsID and matches the single-rsID vignette output
- Output includes taxa-taxa R values before rho adjustment for taxa-SNP association, as described in the paper's rho-estimation method
- Input matrices share sample column names between SnpFile and microbeAbund (rownames SNPs/taxa, colnames sample names)
- Package installed from Bioconductor (https://bioconductor.org/packages/mbQTL/) or GitHub without errors, e.g. BiocManager::install("mbQTL")

## Limitations

- The vignette datasets (SnpFile, microbeAbund) are simulated and are not factual, so results are demonstrations rather than biological findings
- allToAllProduct with linear regression (MatrixEQTL model) is for quantitative taxa abundance; presence/absence (categorical) taxa require the separate logistic regression analysis of one SNP and one taxa
- No changelog/discussion content was available in the README, so version-specific behavior changes are not documented here

## Evidence

- [intro] mbQTL statistical package scope: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
- [intro] rho estimation method: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] linear regression via MatrixEQTL: "A) linear regression analysis which uses core `MatrixeQTL` model."
- [readme] Bioconductor installation: [evidence span withheld — anchor status: external_doc]
