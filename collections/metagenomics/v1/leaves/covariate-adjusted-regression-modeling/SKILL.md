---
name: covariate-adjusted-regression-modeling
description: Use when testing host genetic variants (SNVs/SNPs) for association with
  microbial taxon abundances across a sample cohort and covariates (e.g., covariates.txt/CovFile)
  must be adjusted for in the regression model.
when_to_use_negative:
- Taxa are presence/absence (categorical) rather than continuous abundances — use
  mbQTL's logistic regression approach for one SNP–one taxa relationships instead
- The goal is correlation-based rho estimation accounting for taxa-taxa relationships
  and linkage disequilibrium (LD) SNPs/regions — use mbQTL's rho estimation method
  (Approach B) rather than Approach A linear regression
- Input is a phyloseq or metagenomeSeq object that has not yet been converted to an
  mbQTL-compatible abundance data frame — convert first with phyloseq_to_metagenomeSeq()
  and metagenomeSeqToMbqtl
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3697
tools:
- name: mbQTL
  role: R/Bioconductor package providing linearTaxaSnp(), histPvalueLm(), and qqPlotLm()
    for covariate-adjusted taxa–SNP linear regression and diagnostics
  repo: https://bioconductor.org/packages/mbQTL/
- name: MatrixEQTL
  role: core linear regression model engine underlying mbQTL's taxa–SNP association
    testing (Approach A)
- name: R
  role: runtime environment in which mbQTL is installed (via BiocManager::install
    or devtools::install_github) and executed
  repo: https://github.com/Mercedeh66/mbQTL
- name: metagenomeSeq
  role: provides MRexperiment microbiome objects that can be normalized/aggregated
    and converted to mbQTL-compatible input via metagenomeSeqToMbqtl
- name: phyloseq
  role: source of S4 microbiome objects converted to metagenomeSeq format with phyloseq_to_metagenomeSeq()
    before mbQTL analysis
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:5361c744ab59f7e9dc7342ba4e3c187f46cffa173f5353bbaa1ea84383dfe951
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - knitr
  - mbQTL
  - metagenomeSeq
  - phyloseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# covariate-adjusted-regression-modeling

## Summary

Fits covariate-adjusted linear regressions of each bacterial taxon against each host SNP using mbQTL's linearTaxaSnp(), which wraps the core MatrixEQTL model, to identify taxa–SNV associations with coefficient, P value, FDR, and Bonferroni statistics.

## When to use

Use when testing host genetic variants (SNVs/SNPs) for association with microbial taxon abundances across a sample cohort and covariates (e.g., covariates.txt/CovFile) must be adjusted for in the regression model.

## When NOT to use

- Taxa are presence/absence (categorical) rather than continuous abundances — use mbQTL's logistic regression approach for one SNP–one taxa relationships instead
- The goal is correlation-based rho estimation accounting for taxa-taxa relationships and linkage disequilibrium (LD) SNPs/regions — use mbQTL's rho estimation method (Approach B) rather than Approach A linear regression
- Input is a phyloseq or metagenomeSeq object that has not yet been converted to an mbQTL-compatible abundance data frame — convert first with phyloseq_to_metagenomeSeq() and metagenomeSeqToMbqtl

## Inputs

- microbeAbund: microbial abundance data frame (rownames = taxa, colnames = sample names)
- SnpFile: SNP genotype data frame (rownames = SNPs, colnames = sample names)
- CovFile: optional covariate data frame (rownames = covariates, colnames = sample names), e.g., covariates.txt
- Optionally a metagenomeSeq MRexperiment object or phyloseq S4 object to normalize/convert first

## Outputs

- Linear regression output object with per taxon–SNP pair statistics (coefficient, P value, FDR-adjusted P value, Bonferroni)
- Association statistics table, e.g., linear_taxa_snp_results.tsv
- Diagnostic plots: histogram of P values via histPvalueLm() and QQ-plot of expected versus observed P values via qqPlotLm()

## How to apply

Load mbQTL's example or user datasets: microbeAbund (taxa rows x sample columns), SnpFile (SNP rows x sample columns), and optionally CovFile (covariate rows x sample columns). Call linearTaxaSnp(microbeAbund, SnpFile, Covariate = CovFile), which performs covariate-adjusted linear regression of each taxon against each SNP using the core MatrixEQTL model. Extract the per taxon–SNP pair statistics (coefficient, P value, FDR-adjusted P value) from the returned linear regression output object and write the association table (e.g., linear_taxa_snp_results.tsv). Evaluate the P value distribution with histPvalueLm() and qqPlotLm(), and report FDR and Bonferroni-adjusted significance. Microbiome data may be normalized or aggregated from metagenomeSeq objects via metagenomeSeqToMbqtl, or converted from phyloseq using phyloseq_to_metagenomeSeq(), before running the regression.

## Related tools

- **mbQTL** (R/Bioconductor package providing linearTaxaSnp(), histPvalueLm(), and qqPlotLm() for covariate-adjusted taxa–SNP linear regression and diagnostics) — https://bioconductor.org/packages/mbQTL/
- **MatrixEQTL** (core linear regression model engine underlying mbQTL's taxa–SNP association testing (Approach A))
- **R** (runtime environment in which mbQTL is installed (via BiocManager::install or devtools::install_github) and executed) — https://github.com/Mercedeh66/mbQTL
- **metagenomeSeq** (provides MRexperiment microbiome objects that can be normalized/aggregated and converted to mbQTL-compatible input via metagenomeSeqToMbqtl)
- **phyloseq** (source of S4 microbiome objects converted to metagenomeSeq format with phyloseq_to_metagenomeSeq() before mbQTL analysis)

## Examples

```
data('microbeAbund'); data('SnpFile'); data('CovFile'); res <- linearTaxaSnp(microbeAbund, SnpFile, Covariate = CovFile)
```

## Evaluation signals

- The output association table exists and contains rows for taxon–SNP pairs with both P value and FDR columns populated
- QQ-plot generated by qqPlotLm(microbeAbund, SnpFile, Covariate = CovFile) shows expected versus observed P values consistent with the fitted MatrixEQTL-based model (no gross deviation of observed from expected quantiles)
- histPvalueLm() yields a P value histogram that can be inspected for inflation or enrichment near zero
- Both FDR-adjusted and Bonferroni-adjusted significance reports are produced alongside raw P values
- Sample column names align across microbeAbund, SnpFile, and CovFile so each taxon–SNP regression uses matched samples

## Limitations

- The datasets shipped in the mbQTL vignette are simulated and are not factual, so results on example data are for demonstration only
- Approach A linear regression assumes taxa abundance as a continuous response; for categorical presence/absence taxa, logistic regression is the appropriate mbQTL method instead
- Covariates are optional (Covariate = CovFile); omitting CovFile means no covariate adjustment, which may confound taxa–SNP associations

## Evidence

- [full_text] mbQTL implements Approach A taxa–SNP linear regression through the linearTaxaSnp() function (with optional covariates via Covariate = CovFile): "A) linear regression analysis which uses core `MatrixeQTL` model."
- [intro] Optional CovFile supplies covariates as rownames covariates and colnames sample names: "and SnpFile (rownames snps and colnames sample names) with optional "CovFile""
- [intro] The covariates file is specifically used for the linear regression section: "and the "covariates.txt" file is used for section A)linear regression."
- [full_text] qqPlotLm evaluates the linear regression analysis by plotting expected versus estimated P values for all SNP-taxa tests: [evidence span withheld — anchor status: not_found]
- [readme] mbQTL is installable from Bioconductor and GitHub: [evidence span withheld — anchor status: external_doc]
- [intro] mbQTL can accept normalized metagenomeSeq or converted phyloseq objects as microbiome input: "If one has a `phyloseq` S4 object, they can use the `phyloseq_to_metagenomeSeq()` option of the `phyloseq` package and start from there."
