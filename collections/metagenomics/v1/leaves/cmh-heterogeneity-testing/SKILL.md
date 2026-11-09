---
name: cmh-heterogeneity-testing
description: Use when the analysis goal is to assess adaptation (alpha) for a whole
  gene set — e.g. genes grouped by recombination rate bins or chromosome localization
  — rather than gene-by-gene, and to test heterogeneity of selection among those genes.
when_to_use_negative:
- The goal is to estimate alpha for each individual gene separately — the concatenate
  mode analyzes the selective regime for the whole gene set, not per gene
- The user has only a single gene or a single aggregated 2x2 MKT table and there is
  no per-gene stratification for the CMH statistic
- The question is which of the four MKT variants (standard MKT, FWWMKT, eMKT, aMKT)
  to run on one dataset without a multi-gene grouping hypothesis
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iMKT (R package)
  role: R package that accepts per-gene daf and divergence data frames and provides
    the concatenate-genes mode with CMH heterogeneity testing and bootstrap alpha
    estimation
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web-based service implementing the concatenated-genes analysis for pre-loaded
    D. melanogaster and human protein-coding gene datasets
  repo: https://imkt.uab.cat
- name: Python
  role: Used in the iMKTData Jupyter pipeline to extract the per-gene unfolded SFS
    and divergence counts that feed the multi-gene MKT analysis
  repo: https://github.com/BGD-UAB/iMKTData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:ec3e56ea785f9b1db3a7b404a8289b2bb0c704f316cac817eb60866222f3a9b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGD-UAB/iMKT
  tools:
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cmh-heterogeneity-testing

## Summary

iMKT's multi-gene (concatenate) analysis mode tests whether the selection regime is homogeneous across a user-selected set of protein-coding genes by applying the Cochran-Mantel-Haenszel statistic to per-gene MKT contingency tables, and quantifies uncertainty in the alpha estimate via a 100-replicate gene-resampling bootstrap 95% CI.

## When to use

Use when the analysis goal is to assess adaptation (alpha) for a whole gene set — e.g. genes grouped by recombination rate bins or chromosome localization — rather than gene-by-gene, and to test heterogeneity of selection among those genes.

## When NOT to use

- The goal is to estimate alpha for each individual gene separately — the concatenate mode analyzes the selective regime for the whole gene set, not per gene
- The user has only a single gene or a single aggregated 2x2 MKT table and there is no per-gene stratification for the CMH statistic
- The question is which of the four MKT variants (standard MKT, FWWMKT, eMKT, aMKT) to run on one dataset without a multi-gene grouping hypothesis

## Inputs

- Per-gene unfolded site frequency spectrum / MKT divergence and polymorphism count tables (tab-delimited, Dn, Ds, Pn, Ps) from the iMKTData D. melanogaster pipeline (DGN/PopFly FASTA) or human pipeline (1000GP Phase III VCF)
- A user-defined gene set selection criterion (e.g. recombination rate bins from Comeron et al. 2012, or chromosome localization)

## Outputs

- Concatenated multi-gene MKT table (aggregated Dn, Ds, Pn, Ps counts)
- Cochran-Mantel-Haenszel heterogeneity statistic for the gene set
- Resampling 95% confidence interval for alpha (2.5th and 97.5th percentiles over 100 bootstrap gene-resampling replicates)

## How to apply

Load the per-gene divergence/polymorphism count tables (Dn, Ds, Pn, Ps) derived from D. melanogaster DGN/PopFly FASTA data (or PopHuman VCF data) using the iMKT R package. Concatenate the user-selected gene set into a single multi-gene MKT table and analyze the selective regime for the whole set rather than per gene. Apply the Cochran-Mantel-Haenszel heterogeneity statistic across the per-gene 2x2 MKT contingency tables to test whether the selection regime is homogeneous over the gene set. Run 100 bootstrap replicates sampling genes with replacement, estimating alpha in each replicate with the chosen MK-derived test, then take the 2.5th and 97.5th percentiles of the bootstrap alpha distribution as the resampling 95% confidence interval.

## Related tools

- **iMKT (R package)** (R package that accepts per-gene daf and divergence data frames and provides the concatenate-genes mode with CMH heterogeneity testing and bootstrap alpha estimation) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web-based service implementing the concatenated-genes analysis for pre-loaded D. melanogaster and human protein-coding gene datasets) — https://imkt.uab.cat
- **Python** (Used in the iMKTData Jupyter pipeline to extract the per-gene unfolded SFS and divergence counts that feed the multi-gene MKT analysis) — https://github.com/BGD-UAB/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Output file multi_gene_alpha_results.tsv exists and contains both the CMH heterogeneity statistic and the bootstrap-derived 95% CI fields
- The bootstrap replicate count equals exactly 100 (genes resampled with replacement)
- The 95% CI for alpha is computed as the 2.5th and 97.5th percentiles of the bootstrap alpha distribution
- The per-gene 2x2 MKT contingency tables (neutral vs selected class, polymorphism vs divergence) are correctly stratified by gene before applying the CMH statistic
- The aggregated multi-gene table reports Dn, Ds, Pn and Ps counts consistent with the per-gene input count tables

## Limitations

- In concatenate mode the selection regime is estimated for the whole gene set, so per-gene adaptation rates are not obtained from this analysis
- The bootstrap CI is a resampling (gene-with-replacement) confidence interval and depends on the composition of the user-selected gene set
- iMKT-derived analyses require properly unfolded SFS and divergence counts; the iMKTData pipelines demand correct file paths and downloadable source data (PopFly/DGN FASTA, PopHuman VCF, recombination maps)

## Evidence

- [intro] It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization)"
- [readme] iMKT is an R package to compute the McDonald and Kreitman test on polymorphism and divergence genomic data provided by the user or automatically downloaded from PopFly or PopHuman: [evidence span withheld — anchor status: external_doc]
- [supplementary] The corresponding R package is available for download at https://github.com/BGD-UAB/iMKT: "The corresponding R package is available for download at https://github.com/BGD-UAB/iMKT"
