---
name: logistic-regression-taxa-snp-association-testing
description: Use when testing taxon–SNP associations in 16S/SNP data where the taxon
  is treated as a binary presence/absence (categorical) trait rather than a continuous
  abundance, e.g. with mbQTL's logRegSnpsTaxa.
when_to_use_negative:
- The goal is continuous-abundance taxa–SNP association via linear regression (mbQTL
  Approach A uses the core MatrixEQTL model instead).
- The goal is estimating rho values for taxa–SNP correlation after taxa-taxa R values
  (the correlation/LD approach), rather than categorical presence/absence logistic
  regression.
- The microbe data is a metagenomeSeq or phyloseq object that has not yet been converted
  to an mbQTL-compatible data frame via metagenomeSeqToMbqtl / phyloseq_to_metagenomeSeq().
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3174
tools:
- name: mbQTL
  role: R/Bioconductor package providing the logRegSnpsTaxa function and selectmicrobe
    variant for logistic regression taxa–SNP association testing (Approach C)
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment in which mbQTL's logRegSnpsTaxa is invoked on microbeAbund
    and SnpFile
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:f24220134d4019b3fef480e0e1d7a7fbbe55068e0cdbb6fc650fc21dba602034
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# logistic-regression-taxa-snp-association-testing

## Summary

Approach C of the mbQTL R/Bioconductor package: estimate the relationship of one SNP and one taxa across the sample cohort using logistic regression with presence/absence (categorical) taxa, via the logRegSnpsTaxa function.

## When to use

Use when testing taxon–SNP associations in 16S/SNP data where the taxon is treated as a binary presence/absence (categorical) trait rather than a continuous abundance, e.g. with mbQTL's logRegSnpsTaxa.

## When NOT to use

- The goal is continuous-abundance taxa–SNP association via linear regression (mbQTL Approach A uses the core MatrixEQTL model instead).
- The goal is estimating rho values for taxa–SNP correlation after taxa-taxa R values (the correlation/LD approach), rather than categorical presence/absence logistic regression.
- The microbe data is a metagenomeSeq or phyloseq object that has not yet been converted to an mbQTL-compatible data frame via metagenomeSeqToMbqtl / phyloseq_to_metagenomeSeq().

## Inputs

- microbeAbund: microbial abundance table (rownames taxa, colnames sample names)
- SnpFile: SNP genotype table (rownames SNPs, colnames sample names)
- optional selectmicrobe argument naming a specific taxon (e.g. "Haemophilus")

## Outputs

- Table of significant taxa–SNP logistic regression results (coefficients and statistical measures per taxon–SNP pair)
- logistic_regression_taxa_snp_results.csv

## How to apply

Load the mbQTL simulated example datasets microbeAbund (rownames taxa, colnames samples) and SnpFile (rownames SNPs, colnames samples) with data(microbeAbund) and data(SnpFile). Call logRegSnpsTaxa(microbeAbund, SnpFile) to fit, for each taxon–SNP pair, a logistic regression model using the presence/absence (categorical) taxon status across the sample cohort. To examine a single microbe such as Haemophilus, use the selectmicrobe = "Haemophilus" variant of logRegSnpsTaxa to obtain the one-taxon-vs-one-SNP relationship. Collect the coefficients and statistical measures per pair, and write the significant taxa–SNP logistic regression results to logistic_regression_taxa_snp_results.csv. This is mbQTL's Approach C, distinct from Approach A (MatrixEQTL-based linear regression) and the rho correlation approach.

## Related tools

- **mbQTL** (R/Bioconductor package providing the logRegSnpsTaxa function and selectmicrobe variant for logistic regression taxa–SNP association testing (Approach C)) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment in which mbQTL's logRegSnpsTaxa is invoked on microbeAbund and SnpFile)

## Examples

```
data("microbeAbund"); data("SnpFile"); res <- logRegSnpsTaxa(microbeAbund, SnpFile); res_h <- logRegSnpsTaxa(microbeAbund, SnpFile, selectmicrobe = "Haemophilus")
```

## Evaluation signals

- The output table contains fitted logistic regression statistics for taxon–SNP pairs, including coefficients and statistical measures.
- The Haemophilus-restricted result (via selectmicrobe = "Haemophilus") is present alongside the genome-wide taxon–SNP results.
- The results can be regenerated from the packaged simulated data (microbeAbund, SnpFile) without error.
- The output CSV logistic_regression_taxa_snp_results.csv is written and its schema matches the per-pair statistics table.
- The analysis is restricted to one SNP and one taxa per model, consistent with the one-snp-one-taxa logistic regression design.

## Limitations

- The vignette datasets are simulated and not factual; results from them validate the workflow only.
- Logistic regression treats the taxon as presence/absence (categorical), so continuous abundance information is discarded — use the MatrixEQTL linear regression or rho correlation approaches for abundance-based testing.
- mbQTL integrates 16S rRNA sequencing and SNV/SNP data; other data types require conversion (metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq).

## Evidence

- [intro] Utilizing Logistic regression analysis, we estimate the relationship of one snp and one taxa across the sample cohort: "Utilizing Logistic regression analysis, we estimate the relationship of one snp and one taxa across the sample cohort"
- [other] Approach C logistic regression between taxa and SNPs is performed with logRegSnpsTaxa: [evidence span withheld — anchor status: not_found]
- [intro] microbeAbund and SnpFile input description: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [readme] README install instructions: [evidence span withheld — anchor status: external_doc]
