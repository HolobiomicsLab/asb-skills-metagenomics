---
name: snp-genotype-matrix-processing
description: Use when you have an SNP genotype table with SNPs as rows and sample
  names as columns plus a matching microbial abundance table, and you need to identify
  significant taxa–SNP relationships via logistic regression in R.
when_to_use_negative:
- If the goal is continuous-abundance taxa–SNP association via linear regression,
  use mbQTL's MatrixeQTL-based approach (Approach A) instead of logistic regression
- If the input data are a phyloseq S4 object or metagenomeSeq MRexperiment object,
  first convert/normalize (e.g. phyloseq_to_metagenomeSeq() or metagenomeSeqToMbqtl)
  rather than passing them directly as matrices
- If taxa are measured as continuous abundances rather than presence/absence (categorical)
  status, the logistic regression formulation of Approach C is inappropriate
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: mbQTL
  role: R/Bioconductor package providing logRegSnpsTaxa (Approach C) for logistic
    regression of taxa–SNP relationships, plus the simulated microbeAbund and SnpFile
    example datasets
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment in which the mbQTL package functions are invoked
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:14e0e398dbdd1ec798f1ecfbdfd6ffd3d815a7074cd0918c0600a25275d52f26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# snp-genotype-matrix-processing

## Summary

Formats and uses an SNP genotype matrix (rownames SNPs, colnames samples) together with a microbial abundance matrix as input to mbQTL's Approach C logistic regression (logRegSnpsTaxa), which estimates the relationship of one SNP and one taxa across the sample cohort using presence/absence (categorical) taxa.

## When to use

Use when you have an SNP genotype table with SNPs as rows and sample names as columns plus a matching microbial abundance table, and you need to identify significant taxa–SNP relationships via logistic regression in R.

## When NOT to use

- If the goal is continuous-abundance taxa–SNP association via linear regression, use mbQTL's MatrixeQTL-based approach (Approach A) instead of logistic regression
- If the input data are a phyloseq S4 object or metagenomeSeq MRexperiment object, first convert/normalize (e.g. phyloseq_to_metagenomeSeq() or metagenomeSeqToMbqtl) rather than passing them directly as matrices
- If taxa are measured as continuous abundances rather than presence/absence (categorical) status, the logistic regression formulation of Approach C is inappropriate

## Inputs

- microbeAbund: microbial abundance table with rownames as taxa and colnames as sample names
- SnpFile: SNP genotype table with rownames as SNPs and colnames as sample names
- Optional single taxon name for the selectmicrobe variant (e.g. "Haemophilus")

## Outputs

- Table of significant taxa–SNP logistic regression results with coefficients and statistical measures per pair (saved as logistic_regression_taxa_snp_results.csv)
- One-taxon-vs-one-SNP logistic regression result when using the selectmicrobe variant

## How to apply

Load the simulated example datasets microbeAbund (microbial abundance table, rownames taxa, colnames samples) and SnpFile (SNP genotype table, rownames SNPs, colnames samples) from the mbQTL R/Bioconductor package with data(). Ensure both matrices share aligned sample names in their columns, since mbQTL pairs taxa and SNPs across the same cohort. Invoke logRegSnpsTaxa(microbeAbund, SnpFile) to fit, for each taxon–SNP pair, a logistic regression using the presence/absence (categorical) taxon status. To probe a single taxon–SNP relationship, use the selectmicrobe variant, e.g. selectmicrobe = "Haemophilus", to restrict the model to one taxon. Collect the per-pair logistic regression coefficients and statistical measures (including the Haemophilus-restricted result) and write them to logistic_regression_taxa_snp_results.csv. The rationale is that Approach C treats the taxon as a binary categorical outcome rather than a continuous abundance, complementing mbQTL's linear regression (MatrixeQTL-based) and rho-correlation approaches.

## Related tools

- **mbQTL** (R/Bioconductor package providing logRegSnpsTaxa (Approach C) for logistic regression of taxa–SNP relationships, plus the simulated microbeAbund and SnpFile example datasets) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment in which the mbQTL package functions are invoked)

## Examples

```
library(mbQTL); data(microbeAbund); data(SnpFile); res <- logRegSnpsTaxa(microbeAbund, SnpFile)
```

## Evaluation signals

- Output table contains fitted logistic regression statistics for taxon–SNP pairs, not linear regression or correlation coefficients
- The Haemophilus-restricted result from the selectmicrobe variant appears in the output
- The analysis regenerates from the packaged simulated data (data("microbeAbund"); data("SnpFile")) without error
- Results are reproducibly written to logistic_regression_taxa_snp_results.csv with one row per tested taxon–SNP pair
- Both input matrices share consistent sample names across columns

## Limitations

- The vignette datasets (microbeAbund, SnpFile) are simulated and not factual; the 'Taxa_table_MVP_cor.txt' and 'SNP_table_MVP_cor.txt' files are used in all three analysis sections
- The logistic regression approach estimates the relationship of only one SNP and one taxa at a time (presence/absence categorical taxa), not joint multi-locus models
- Covariates (CovFile) are documented for the linear regression section, not stated as supported for the logistic regression call logRegSnpsTaxa(microbeAbund, SnpFile)

## Evidence

- [intro] Utilizing Logistic regression analysis, we estimate the relationship of one snp and one taxa across the sample cohort: "Utilizing Logistic regression analysis, we estimate the relationship of one snp and one taxa across the sample cohort"
- [intro] SnpFile with optional CovFile: "and SnpFile (rownames snps and colnames sample names) with optional "CovFile""
- [methods] data("SnpFile"): [evidence span withheld — anchor status: not_found]
- [intro] mbQTL statistical package purpose: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
- [abstract] Bioconductor availability: "mbQTL is available on bioconductor at https://bioconductor.org/packages/mbQTL/."
