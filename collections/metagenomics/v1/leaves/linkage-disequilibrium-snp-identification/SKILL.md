---
name: linkage-disequilibrium-snp-identification
description: Use when you need to identify taxa associated with SNPs while jointly
  controlling correlation (rho) and goodness-of-fit (R-squared) measures, e.g. flagging
  LD SNPs/regions from 16S taxa abundance and genotype matrices.
when_to_use_negative:
- Your taxa are presence/absence (categorical) — use mbQTL's logistic regression analysis
  for one SNP vs one taxa instead.
- You want linear-regression mbQTL testing per the core MatrixEQTL model (Approach
  A), not correlation-based rho estimation adjusted for taxa-taxa R.
- You lack taxa abundance data or a taxa x sample matrix from which coringTaxa R values
  can be estimated.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: mbQTL
  role: Provides coringTaxa, allToAllProduct, and taxaSnpCor functions used to estimate
    taxa-taxa R values and adjusted taxa-SNP rho values for LD identification (Approach
    B)
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment in which the mbQTL package functions are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:e952235214f82483f5fa38aad9659e83c43f25c7c8fad4c79182f54b7aa0a5eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# linkage-disequilibrium-snp-identification

## Summary

Identifies host-genetics-associated taxa and flags linkage-disequilibrium (LD) SNPs/regions using mbQTL's Approach B: rho values for taxa-SNP correlation are estimated after taxa-taxa R values are estimated, then adjusted for taxa-SNP association via coringTaxa, allToAllProduct, and taxaSnpCor.

## When to use

Use when you need to identify taxa associated with SNPs while jointly controlling correlation (rho) and goodness-of-fit (R-squared) measures, e.g. flagging LD SNPs/regions from 16S taxa abundance and genotype matrices.

## When NOT to use

- Your taxa are presence/absence (categorical) — use mbQTL's logistic regression analysis for one SNP vs one taxa instead.
- You want linear-regression mbQTL testing per the core MatrixEQTL model (Approach A), not correlation-based rho estimation adjusted for taxa-taxa R.
- You lack taxa abundance data or a taxa x sample matrix from which coringTaxa R values can be estimated.

## Inputs

- SnpFile: data frame of SNP genotypes (rownames SNPs, colnames sample names)
- microbeAbund: data frame of microbial/taxa abundance (rownames taxa, colnames sample names)
- Optional CovFile: data frame of covariates (rownames covariates, colnames sample names)

## Outputs

- coringTaxa output: estimated taxa-taxa R values across the sample cohort
- allToAllProduct output: extreme taxa-taxa correlations retained at probs = c(0.0001, 0.9999)
- rho estimates table for all SNP-taxa pairs (taxa, SNP, rho) written to rho_estimates.tsv

## How to apply

Load the mbQTL R/Bioconductor package and attach the vignette-style inputs SnpFile (rownames SNPs, colnames samples) and microbeAbund (rownames taxa, colnames samples) via data(). Run coringTaxa to estimate taxa-taxa correlation (R) values across the sample cohort. Pass the coringTaxa output to allToAllProduct with the extreme-quantile variant probs = c(0.0001, 0.9999) to retain only the most extreme taxa-taxa correlations. Then run taxaSnpCor on the coringTaxa and allToAllProduct outputs to estimate rho values for taxa-SNP correlation adjusted for the taxa-taxa R values. Jointly evaluate rho and R-squared measures to find clusters of taxa associated with SNPs and flag potential LD SNPs/regions, per Approach B.

## Related tools

- **mbQTL** (Provides coringTaxa, allToAllProduct, and taxaSnpCor functions used to estimate taxa-taxa R values and adjusted taxa-SNP rho values for LD identification (Approach B)) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment in which the mbQTL package functions are executed)

## Examples

```
data("SnpFile"); data("microbeAbund"); cor <- coringTaxa(microbeAbund); prod <- allToAllProduct(cor, probs = c(0.0001, 0.9999)); rho <- taxaSnpCor(cor, prod)
```

## Evaluation signals

- rho_estimates.tsv exists and is a well-formed table with taxa, SNP, and rho estimate columns.
- The rho values reproduce the taxa-SNP correlations adjusted for the taxa-taxa R values estimated by coringTaxa, as described for mbQTL's Approach B.
- The allToAllProduct step used probs = c(0.0001, 0.9999), retaining only the most extreme taxa-taxa correlations.
- SNPs/regions flagged as LD are identified only where taxa-SNP associations are jointly supported by both rho and R-squared measures.

## Limitations

- The vignette datasets used by the paper (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt, microbeAbund, SnpFile, CovFile) are simulated and are not factual.
- Approach B requires prior estimation and adjustment for taxa-taxa R values; it cannot produce adjusted rho estimates without the coringTaxa step.

## Evidence

- [intro] rho values estimated after taxa-taxa R value estimation and adjustment: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] jointly evaluating and controlling both measures to find taxa-SNP clusters: "we find clusters of taxa associated with various SNPs across the dataset by evaluating and controlling for both these measures together"
- [intro] mbQTL scope statement: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
- [readme] installation route grounding the R invocation: [evidence span withheld — anchor status: external_doc]
