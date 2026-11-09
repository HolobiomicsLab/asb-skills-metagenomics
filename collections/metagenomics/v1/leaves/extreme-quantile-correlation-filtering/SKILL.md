---
name: extreme-quantile-correlation-filtering
description: 'Use when running mbQTL''s Approach B: you have estimated taxa-taxa correlation
  (R) values with coringTaxa and need to retain only the most extreme pairwise correlations
  before estimating taxa-SNP rho values with taxaSnpCor.'
when_to_use_negative:
- Running mbQTL Approach A (linear regression via the MatrixeQTL core model) or the
  logistic regression analysis for presence/absence (categorical) taxa — these do
  not involve taxa-taxa R filtering.
- Input is not taxa-taxa correlation output but raw abundance or genotype data — coringTaxa
  must be run first.
- The goal is unadjusted pairwise correlation exploration without the taxa-SNP association
  step; the extreme-quantile filter exists specifically to support adjusted rho estimation
  in Approach B.
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: mbQTL
  role: Provides coringTaxa, allToAllProduct, and taxaSnpCor used for the quantile
    filtering and subsequent rho estimation
  repo: https://github.com/Mercedeh66/mbQTL
- name: R
  role: Execution environment; mbQTL is an R/Bioconductor package installed via BiocManager
  repo: https://bioconductor.org/packages/mbQTL/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:bdf28ca8deca4bfb61a114eb41d33e035548de12a0d9de6d5ae5c8a016f4bb79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mercedeh66/mbQTL
  tools:
  - R
  - mbQTL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# extreme-quantile-correlation-filtering

## Summary

A filtering step from the mbQTL Approach B workflow in which all pairwise taxa-taxa correlations (R values, estimated with coringTaxa) are reduced to only the most extreme associations using allToAllProduct with probs = c(0.0001, 0.9999), so that subsequent taxa-SNP rho estimation (taxaSnpCor) is adjusted for extreme taxa-taxa correlation structure and can flag linkage disequilibrium (LD) SNPs/regions.

## When to use

Use when running mbQTL's Approach B: you have estimated taxa-taxa correlation (R) values with coringTaxa and need to retain only the most extreme pairwise correlations before estimating taxa-SNP rho values with taxaSnpCor.

## When NOT to use

- Running mbQTL Approach A (linear regression via the MatrixeQTL core model) or the logistic regression analysis for presence/absence (categorical) taxa — these do not involve taxa-taxa R filtering.
- Input is not taxa-taxa correlation output but raw abundance or genotype data — coringTaxa must be run first.
- The goal is unadjusted pairwise correlation exploration without the taxa-SNP association step; the extreme-quantile filter exists specifically to support adjusted rho estimation in Approach B.

## Inputs

- microbeAbund-style abundance data frame (rownames taxa, colnames sample names) loaded via data("microbeAbund")
- Output of coringTaxa (taxa-taxa correlation R values)

## Outputs

- allToAllProduct output containing only the most extreme taxa-taxa correlations (probs = c(0.0001, 0.9999))
- taxaSnpCor rho estimates for all SNP-taxa pairs, written to rho_estimates.tsv with taxa, SNP, and rho estimate columns

## How to apply

First run coringTaxa on the microbial abundance data (microbeAbund, rownames taxa, colnames sample names) to estimate taxa-taxa correlation (R) values across the sample cohort. Then pass the coringTaxa output to allToAllProduct using the probs = c(0.0001, 0.9999) quantile variant, which retains only the most extreme taxa-taxa correlations at both tails of the distribution. Finally pass both the coringTaxa and filtered allToAllProduct outputs to taxaSnpCor so rho values for all SNP-taxa pairs are estimated adjusted for the previously estimated taxa-taxa R values. The rationale, per mbQTL's Approach B, is to jointly evaluate rho and R-squared goodness-of-fit measures so that clusters of taxa associated with SNPs can be identified while controlling for both measures together and flagging potential LD SNPs/regions.

## Related tools

- **mbQTL** (Provides coringTaxa, allToAllProduct, and taxaSnpCor used for the quantile filtering and subsequent rho estimation) — https://github.com/Mercedeh66/mbQTL
- **R** (Execution environment; mbQTL is an R/Bioconductor package installed via BiocManager) — https://bioconductor.org/packages/mbQTL/

## Examples

```
R <- coringTaxa(microbeAbund); extreme <- allToAllProduct(R, probs = c(0.0001, 0.9999))
```

## Evaluation signals

- rho_estimates.tsv exists and is a well-formed table with taxa, SNP, and rho estimate columns
- Rho values reproduce the taxa-SNP correlations adjusted for taxa-taxa R values as described for mbQTL's Approach B
- The allToAllProduct output contains only taxa-taxa pairs falling at or beyond the 0.0001 and 0.9999 quantiles of the correlation distribution
- taxaSnpCor is called with both the coringTaxa output and the filtered allToAllProduct output, not with raw abundance data

## Limitations

- The vignette datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt) are simulated and not factual, so results reproduce methodology rather than biological findings
- The extreme quantile cutoff (probs = c(0.0001, 0.9999)) retains very few taxa-taxa pairs; adjusting quantiles changes which correlations survive and downstream rho adjustment
- The filtered taxa-taxa R values are used to flag potential linkage disequilibrium (LD) SNPs/regions; without this filter, taxa-SNP associations may be confounded by correlated taxa structure

## Evidence

- [intro] Estimate rho values for taxa-SNP correlation after estimating taxa-taxa R values, allowing simultaneous estimation of taxa-SNP associations and identification of linkage disequilibrium (LD) SNPs and regions: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] Jointly evaluate rho and R-squared measures to identify taxa-SNP associations and flag potential LD SNPs/regions per Approach B: "we find clusters of taxa associated with various SNPs across the dataset by evaluating and controlling for both these measures together"
- [readme] mbQTL is available on Bioconductor: [evidence span withheld — anchor status: external_doc]
