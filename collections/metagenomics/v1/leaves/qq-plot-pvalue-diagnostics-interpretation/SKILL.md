---
name: qq-plot-pvalue-diagnostics-interpretation
description: Use when a user has run or plans mbQTL Approach A (linear regression
  taxa-SNP testing) and needs to assess overall model performance/inflation of the
  estimated P values before interpreting associations.
when_to_use_negative:
- The taxa are presence/absence (categorical) and the relationship of one SNP and
  one taxa is estimated via mbQTL's logistic regression approach rather than Approach
  A linear regression.
- The P values to diagnose do not come from mbQTL's MatrixEQTL-based linear regression
  taxa-SNP model (e.g. they come from a different GWAS/QTL tool).
- The input microbiome data is still in metagenomeSeq (MRexperiment) or phyloseq S4
  format and has not yet been converted to an mbQTL-compatible data frame.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mbQTL
  role: Provides the qqPlotLm function that fits the MatrixEQTL-based linear regression
    taxa-SNP model and draws the QQ-plot of expected versus observed P values
  repo: https://bioconductor.org/packages/mbQTL/
- name: MatrixEQTL
  role: Core linear regression model underlying the taxa-SNP estimations whose P values
    are evaluated in the QQ-plot
- name: R
  role: Runtime environment in which mbQTL and the vignette workflow (data(), qqPlotLm())
    are executed
- name: knitr
  role: Vignette engine used to render the documentation demonstrating qqPlotLm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:5104d9e02f500002fb08177dad55334d61fc497859848b0523066bd54979bbcc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - knitr
  - mbQTL
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# qq-plot-pvalue-diagnostics-interpretation

## Summary

Diagnostic skill for evaluating mbQTL's Approach A (MatrixEQTL-based linear regression taxa-SNP association testing) by rendering a QQ-plot of expected versus estimated P values across all SNP-taxa linear regressions, using the qqPlotLm function.

## When to use

Use when a user has run or plans mbQTL Approach A (linear regression taxa-SNP testing) and needs to assess overall model performance/inflation of the estimated P values before interpreting associations.

## When NOT to use

- The taxa are presence/absence (categorical) and the relationship of one SNP and one taxa is estimated via mbQTL's logistic regression approach rather than Approach A linear regression.
- The P values to diagnose do not come from mbQTL's MatrixEQTL-based linear regression taxa-SNP model (e.g. they come from a different GWAS/QTL tool).
- The input microbiome data is still in metagenomeSeq (MRexperiment) or phyloseq S4 format and has not yet been converted to an mbQTL-compatible data frame.

## Inputs

- microbeAbund: microbial abundance data frame (rownames taxa, colnames sample names)
- SnpFile: SNP genotype data frame (rownames SNPs, colnames sample names)
- CovFile (optional): covariate data frame (rownames covariates, colnames sample names)

## Outputs

- QQ-plot figure of expected versus estimated P values for all SNP-taxa linear regression estimations (saved as qqPlotLm.png at 300 dpi)

## How to apply

Load the mbQTL package in R/Bioconductor and load the three packaged example datasets with data(): microbeAbund (microbial abundance, taxa rows x sample columns), SnpFile (SNP genotypes, SNP rows x sample columns), and CovFile (covariates, covariate rows x sample columns). Call qqPlotLm(microbeAbund, SnpFile, Covariate = CovFile); the function fits the MatrixEQTL-based linear regression taxa-SNP model and draws the QQ-plot of expected versus observed (estimated) P values for all SNP-taxa linear regression estimations. Judge application success by the rendered figure being produced and displaying observed P values against the expected diagonal, enabling evaluation of Approach A performance; save the figure at 300 dpi as qqPlotLm.png. Note that the packaged vignette datasets are simulated, so the diagnostic pattern — not specific P value patterns — is what should be transferred to real 16S rRNA/SNP datasets.

## Related tools

- **mbQTL** (Provides the qqPlotLm function that fits the MatrixEQTL-based linear regression taxa-SNP model and draws the QQ-plot of expected versus observed P values) — https://bioconductor.org/packages/mbQTL/
- **MatrixEQTL** (Core linear regression model underlying the taxa-SNP estimations whose P values are evaluated in the QQ-plot)
- **R** (Runtime environment in which mbQTL and the vignette workflow (data(), qqPlotLm()) are executed)
- **knitr** (Vignette engine used to render the documentation demonstrating qqPlotLm)

## Examples

```
data("microbeAbund"); data("SnpFile"); data("CovFile"); qqPlotLm(microbeAbund, SnpFile, Covariate = CovFile)
```

## Evaluation signals

- A rendered QQ-plot figure is produced showing observed versus expected P values on comparable axes
- The plot reflects P values from all SNP-taxa linear regression estimations fitted by the MatrixEQTL-based model, not a subset
- The output figure is saved as qqPlotLm.png at 300 dpi
- All three input objects (microbeAbund, SnpFile, CovFile) have taxa/SNP/covariate rows and matching sample-name columns before the call
- Observed P values approximately follow the expected diagonal (no gross deviation) for a well-behaved Approach A analysis

## Limitations

- The vignette datasets used to demonstrate the QQ-plot (microbeAbund, SnpFile, CovFile) are all simulated and are not factual.
- This QQ-plot diagnostic applies only to Approach A (linear regression, MatrixEQTL core); it does not cover the logistic regression presence/absence approach or the rho/correlation-based taxa-SNP estimation.
- CovFile is an optional input for the linear regression section; omitting covariates may confound taxa-SNP association estimates.

## Evidence

- [intro] mbQTL statistical package scope: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
- [intro] Approach A linear regression model: "A) linear regression analysis which uses core `MatrixeQTL` model."
- [intro] Input file definitions: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [intro] Simulated vignette data caveat: "The datasets used for this vignette are all simulated and are not factual."
- [readme] Installation from GitHub: [evidence span withheld — anchor status: external_doc]
