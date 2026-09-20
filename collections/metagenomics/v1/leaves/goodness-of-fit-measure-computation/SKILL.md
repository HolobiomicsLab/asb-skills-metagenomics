---
name: goodness-of-fit-measure-computation
description: Reference for computing pairwise taxa–taxa correlation strength (R) and
  per-taxa goodness-of-fit (R²) from a taxa-by-sample 16S abundance data frame, before
  estimating rho for taxa–SNP correlation in mbQTL.
when_to_use_negative:
- Input is already a correlation matrix or SNP–taxa R² table rather than raw taxa
  abundance data — proceed directly to the rho estimation step instead
- Taxa are categorical presence/absence values — mbQTL specifies logistic regression
  analysis for that case, not correlation-based Approach B
- The goal is linear regression-based taxa–SNP association (the core MatrixeQTL model
  section), which does not require the taxa–taxa correlation step
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: mbQTL
  role: provides coringTaxa() to compute taxa–taxa correlation (R) and goodness-of-fit
    (R²) measures feeding rho estimation
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: runtime environment in which mbQTL and the coringTaxa call are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:cf2d4194b335a8b40bc53095c1da538cc611633bcfe54a479e04a0244a0e44dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# goodness-of-fit-measure-computation

## Summary

Compute taxa–taxa correlation (R) and goodness-of-fit (R²) measures in mbQTL via coringTaxa, producing the correlation matrix that is a prerequisite input to mbQTL's Approach B rho estimation for taxa–SNP associations.

## When to use

Reference for computing pairwise taxa–taxa correlation strength (R) and per-taxa goodness-of-fit (R²) from a taxa-by-sample 16S abundance data frame, before estimating rho for taxa–SNP correlation in mbQTL.

## When NOT to use

- Input is already a correlation matrix or SNP–taxa R² table rather than raw taxa abundance data — proceed directly to the rho estimation step instead
- Taxa are categorical presence/absence values — mbQTL specifies logistic regression analysis for that case, not correlation-based Approach B
- The goal is linear regression-based taxa–SNP association (the core MatrixeQTL model section), which does not require the taxa–taxa correlation step

## Inputs

- taxa-by-sample microbial abundance data frame (rownames taxa, colnames sample names), e.g. the packaged microbeAbund example data in mbQTL

## Outputs

- correlationMicrobes object: a square taxa-by-taxa table of correlation (R) values
- one goodness-of-fit (R²) entry per taxon pair
- optional exported table, e.g. coringTaxa_R_table.csv

## How to apply

Load the mbQTL package in R and attach the packaged example abundance data with data("microbeAbund"), a taxa-by-sample data frame with rownames as taxa and colnames as sample names. Call coringTaxa(microbeAbund) to obtain the taxa–taxa correlation matrix (R values) together with the associated goodness-of-fit (R²) measures, writing the result to e.g. coringTaxa_R_table.csv. These measures are one of the two ingredients — alongside SNP–taxa R² — that mbQTL combines via taxaSnpCor to 'estimate rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP', enabling clusters of taxa associated with SNPs to be found 'by evaluating and controlling for both these measures together'.

## Related tools

- **mbQTL** (provides coringTaxa() to compute taxa–taxa correlation (R) and goodness-of-fit (R²) measures feeding rho estimation) — https://bioconductor.org/packages/mbQTL/
- **R** (runtime environment in which mbQTL and the coringTaxa call are executed)

## Examples

```
data(microbeAbund); correlationMicrobes <- coringTaxa(microbeAbund)
```

## Evaluation signals

- The returned correlation table is square (taxa-by-taxa) in structure
- Off-diagonal correlation values fall within [-1, 1]
- Exactly one R² (goodness-of-fit) entry is present per taxon pair
- The output structure matches what mbQTL requires before its rho estimation step (i.e., combinable with SNP–taxa R² via taxaSnpCor)
- Reproducing the vignette call coringTaxa(microbeAbund) on the packaged example data yields the expected correlationMicrobes object

## Limitations

- The vignette datasets used to demonstrate this step are simulated and 'are not factual', so absolute R/R² magnitudes from examples are illustrative only
- The correlation/goodness-of-fit route applies to continuous taxa abundance data; presence/absence (categorical) taxa are handled via logistic regression in a separate mbQTL section
- R and R² measures alone do not identify taxa–SNP relationships — they must be combined with SNP–taxa measures to find clusters of taxa associated with SNPs

## Evidence

- [intro] rho estimation is a novel method requiring taxa–taxa R first: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] both measures are evaluated together to find taxa clusters associated with SNPs: "we find clusters of taxa associated with various SNPs across the dataset by evaluating and controlling for both these measures together"
- [intro] abundance input format for mbQTL functions: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [readme] mbQTL installation via Bioconductor: [evidence span withheld — anchor status: external_doc]
