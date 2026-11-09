---
name: microbe-snp-linear-regression-qtl-analysis
description: Use when taxa abundance and SNP genotype matrices over the same samples
  need taxa-SNP association testing via linear regression, and model-wide P value
  behavior must be checked with a QQ-plot.
when_to_use_negative:
- Taxa are presence/absence (categorical) rather than continuous abundance — mbQTL's
  logistic regression analysis is the appropriate approach for that case
- The goal is taxa-SNP correlation with rho/LD estimation rather than the MatrixEQTL
  linear regression model (Approach A)
- Input is a phyloseq S4 or metagenomeSeq MRexperiment object that first needs conversion/normalization
  via phyloseq_to_metagenomeSeq() or metagenomeSeqToMbqtl before the regression can
  run
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0091
tools:
- name: mbQTL
  role: R/Bioconductor package providing the taxa-SNP linear regression model (Approach
    A) and qqPlotLm for QQ-plot evaluation
  repo: https://bioconductor.org/packages/mbQTL/
- name: MatrixEQTL
  role: Core model underlying mbQTL's linear regression analysis (Approach A) for
    taxa-SNP association testing
- name: R
  role: Runtime environment in which mbQTL is loaded, packaged datasets are accessed
    with data(), and the QQ-plot is rendered
- name: knitr
  role: Vignette engine used to render the mbQTL documentation demonstrating qqPlotLm
    usage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:ed3ff59f411f0d1a6fe65586f8e607211345d68904f7dc79cd4fbb2ba79d739e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - knitr
  - mbQTL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# microbe-snp-linear-regression-qtl-analysis

## Summary

mbQTL's Approach A fits a MatrixEQTL-based linear regression model to test all taxa-SNP associations across a sample cohort, then evaluates model behavior via a QQ-plot of expected versus observed P values. This lets users estimate microbial quantitative trait loci (mbQTLs) and gauge genome-wide inflation of the linear regression P values.

## When to use

Use when taxa abundance and SNP genotype matrices over the same samples need taxa-SNP association testing via linear regression, and model-wide P value behavior must be checked with a QQ-plot.

## When NOT to use

- Taxa are presence/absence (categorical) rather than continuous abundance — mbQTL's logistic regression analysis is the appropriate approach for that case
- The goal is taxa-SNP correlation with rho/LD estimation rather than the MatrixEQTL linear regression model (Approach A)
- Input is a phyloseq S4 or metagenomeSeq MRexperiment object that first needs conversion/normalization via phyloseq_to_metagenomeSeq() or metagenomeSeqToMbqtl before the regression can run

## Inputs

- microbeAbund: microbial abundance data frame (rownames = taxa, colnames = sample names)
- SnpFile: SNP genotype data frame (rownames = snps, colnames = sample names)
- CovFile: optional covariate data frame (rownames = covariates, colnames = sample names)

## Outputs

- Linear regression taxa-SNP association results (P values for all SNP-taxa estimations via the MatrixEQTL model)
- QQ-plot of expected versus estimated P values for Approach A (rendered, e.g. saved as qqPlotLm.png at 300 dpi)

## How to apply

Load the mbQTL package in R/Bioconductor and load its three packaged example datasets with data(): microbeAbund (microbial abundance, rownames taxa x colnames sample names), SnpFile (SNP genotypes, rownames snps x colnames sample names), and the optional CovFile (covariates, rownames covariates x colnames sample names). Call the MatrixEQTL-based linear regression routine with these inputs to estimate all SNP-taxa relationships across the sample cohort, optionally adjusting for covariates. To judge whether Approach A is performing well, call qqPlotLm(microbeAbund, SnpFile, Covariate = CovFile), which fits the same linear regression model and draws a QQ-plot of expected versus estimated P values for all SNP-taxa tests. A QQ-plot tracking the expected line indicates well-calibrated P values, while systematic deviation signals inflation or confounding; save the rendered figure at 300 dpi as qqPlotLm.png for the record.

## Related tools

- **mbQTL** (R/Bioconductor package providing the taxa-SNP linear regression model (Approach A) and qqPlotLm for QQ-plot evaluation) — https://bioconductor.org/packages/mbQTL/
- **MatrixEQTL** (Core model underlying mbQTL's linear regression analysis (Approach A) for taxa-SNP association testing)
- **R** (Runtime environment in which mbQTL is loaded, packaged datasets are accessed with data(), and the QQ-plot is rendered)
- **knitr** (Vignette engine used to render the mbQTL documentation demonstrating qqPlotLm usage)

## Examples

```
library(mbQTL); data("microbeAbund"); data("SnpFile"); data("CovFile"); qqPlotLm(microbeAbund, SnpFile, Covariate = CovFile)
```

## Evaluation signals

- qqPlotLm successfully fits the MatrixEQTL linear regression model and renders a QQ-plot comparing expected versus estimated P values for all SNP-taxa tests
- Input dimensions follow the documented convention: microbeAbund has taxa as rownames and sample names as colnames, SnpFile has SNPs as rownames, CovFile has covariates as rownames
- Datasets load cleanly via data("microbeAbund"), data("SnpFile"), and data("CovFile") from the packaged mbQTL example data
- The QQ-plot figure is exported at 300 dpi as qqPlotLm.png
- Observed P values in the QQ-plot fall near the expected line, indicating no gross inflation of the linear regression P values

## Limitations

- The packaged example datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt, covariates.txt) are simulated and are not factual, so results are for demonstration only
- CovFile is optional and applies to the linear regression section (Approach A) only; other mbQTL analyses do not use it
- README provides no changelog or discussion of known issues ('_No changelog found._'), so edge-case behavior must be verified empirically

## Evidence

- [intro] mbQTL identifies taxa-SNP relationships via linear regression: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
- [intro] Approach A uses the core MatrixEQTL linear regression model: "A) linear regression analysis which uses core `MatrixeQTL` model."
- [intro] Input data conventions for abundance and SNP files: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [readme] mbQTL is installed from Bioconductor via BiocManager: [evidence span withheld — anchor status: external_doc]
