---
name: multiple-testing-fdr-correction
description: Use when a taxa–SNP regression sweep (e.g., mbQTL linearTaxaSnp over
  microbeAbund × SnpFile) yields thousands of P values that must be corrected for
  multiple comparisons and diagnostic-checked before calling mbQTL associations.
when_to_use_negative:
- Taxa are being analyzed as presence/absence (categorical) variables — mbQTL uses
  logistic regression for that case, not the linear-regression/FDR workflow
- The goal is correlation-based estimation of rho values for taxa–SNP association
  and linkage disequilibrium (Approach B-type analysis), rather than linear regression
  testing
- Only one taxon–SNP hypothesis is tested, so no multiple-testing correction is needed
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: mbQTL
  role: Provides linearTaxaSnp() for the underlying taxon–SNP linear regression, FDR
    and Bonferroni reporting, and histPvalueLm()/qqPlotLm() P value diagnostics
  repo: https://bioconductor.org/packages/mbQTL/
- name: MatrixEQTL
  role: Core model engine that linearTaxaSnp() uses to compute the taxon–SNP linear
    regression statistics
- name: R
  role: Execution environment for running mbQTL and its diagnostic plotting functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:73958e69e8a81ba2dc15a6ca43b3d52236692178997465b9e00eb39959194fac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multiple-testing-fdr-correction

## Summary

Applies FDR (and Bonferroni) multiple-testing correction to taxa–SNP association results from mbQTL's linearTaxaSnp(), and evaluates the resulting P value distributions with histPvalueLm() and qqPlotLm() so that significant microbe–SNV calls are statistically valid across the large number of tested taxon–SNP pairs.

## When to use

Use when a taxa–SNP regression sweep (e.g., mbQTL linearTaxaSnp over microbeAbund × SnpFile) yields thousands of P values that must be corrected for multiple comparisons and diagnostic-checked before calling mbQTL associations.

## When NOT to use

- Taxa are being analyzed as presence/absence (categorical) variables — mbQTL uses logistic regression for that case, not the linear-regression/FDR workflow
- The goal is correlation-based estimation of rho values for taxa–SNP association and linkage disequilibrium (Approach B-type analysis), rather than linear regression testing
- Only one taxon–SNP hypothesis is tested, so no multiple-testing correction is needed

## Inputs

- microbeAbund: microbial abundance data frame (rownames = taxa, colnames = sample names)
- SnpFile: SNP genotype data frame (rownames = SNPs, colnames = sample names)
- CovFile (optional): covariate data frame (rownames = covariates, colnames = sample names)

## Outputs

- Linear regression output object with per taxon–SNP pair coefficient, P value, and FDR-adjusted P value, plus Bonferroni reporting
- P value diagnostic plots from histPvalueLm() (histogram) and qqPlotLm() (Q–Q plot)
- Association statistics table, e.g., linear_taxa_snp_results.tsv with taxon–SNP rows and populated P value and FDR columns

## How to apply

Run covariate-adjusted linear regression of each taxon against each SNP with linearTaxaSnp(microbeAbund, SnpFile, Covariate = CovFile), which uses the core MatrixEQTL model. From the returned object, extract the per taxon–SNP pair statistics (coefficient, P value, FDR-adjusted P value) alongside Bonferroni reporting. Because the number of tests scales as taxa × SNPs, rely on the FDR-adjusted P value (not the raw P value) to declare a taxon–SNP pair significant. Validate the raw P values' behavior with histPvalueLm() and qqPlotLm(): a flat P value histogram and a well-behaved Q–Q plot indicate the model and null distribution are appropriate before interpreting FDR calls. Write the association table (e.g., linear_taxa_snp_results.tsv) with both P value and FDR columns populated as the deliverable.

## Related tools

- **mbQTL** (Provides linearTaxaSnp() for the underlying taxon–SNP linear regression, FDR and Bonferroni reporting, and histPvalueLm()/qqPlotLm() P value diagnostics) — https://bioconductor.org/packages/mbQTL/
- **MatrixEQTL** (Core model engine that linearTaxaSnp() uses to compute the taxon–SNP linear regression statistics)
- **R** (Execution environment for running mbQTL and its diagnostic plotting functions)

## Examples

```
out <- linearTaxaSnp(microbeAbund, SnpFile, Covariate = CovFile); histPvalueLm(out); qqPlotLm(out)
```

## Evaluation signals

- Output table contains rows for every tested taxon–SNP pair with both a raw P value column and an FDR-adjusted P value column populated
- FDR-adjusted P values are monotonically non-decreasing with the raw P values and are ≥ the raw P values
- histPvalueLm() histogram of raw P values shows the expected flat profile under the null, and qqPlotLm() shows no gross deviation from the expected line
- Re-running linearTaxaSnp() with the same microbeAbund, SnpFile, and CovFile inputs reproduces the same coefficients, P values, and FDR values
- Bonferroni-reported thresholds are consistent with the total number of taxon–SNP tests performed

## Limitations

- The vignette datasets are simulated and not factual, so results on bundled data should not be interpreted biologically
- FDR and Bonferroni reporting is tied to the linear regression (Approach A) output; other mbQTL approaches (rho estimation, logistic regression) require their own correction strategy
- P value diagnostics (histPvalueLm(), qqPlotLm()) are qualitative checks; a mis-specified model or unaccounted covariates can invalidate FDR calls even if the table is well formed

## Evidence

- [abstract] We describe various statistical methods implemented for the identification of microbe-SNV pairs, relevant statistical measures, and plot functionality for interpretation.: "We describe various statistical methods implemented for the identification of microbe-SNV pairs, relevant statistical measures, and plot functionality for interpretation."
- [intro] linear regression analysis which uses core MatrixEQTL model: "A) linear regression analysis which uses core `MatrixeQTL` model."
- [other] histPvalueLm() and qqPlotLm() P value evaluation alongside FDR and Bonferroni reporting: [evidence span withheld — anchor status: not_found]
- [intro] simulated vignette datasets disclaimer: "The datasets used for this vignette are all simulated and are not factual."
