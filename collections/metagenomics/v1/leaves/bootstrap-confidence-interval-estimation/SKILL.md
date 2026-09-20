---
name: bootstrap-confidence-interval-estimation
description: Use when a gene set (e.g. selected by recombination rate bins or chromosome
  localization) needs a confidence interval on alpha rather than a single point estimate,
  and when homogeneity of selection across genes must be statistically assessed.
when_to_use_negative:
- The goal is a per-gene alpha estimate for individual genes rather than a gene-set-level
  regime and CI — in concatenate mode iMKT analyzes the whole gene set rather than
  each gene separately
- Input data are not polymorphism/divergence MKT counts (e.g. raw FASTA or VCF without
  the iMKTData pipeline's unfolded site frequency spectrum table having been produced
  first)
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: iMKT (R package)
  role: Performs the MK-derived tests to estimate alpha in each bootstrap replicate
    and provides the concatenate-mode analysis with CMH heterogeneity test and resampling
    CI
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web service offering the multi-gene 'Concatenate genes' mode for testing heterogeneity
    of selection and bootstrap CIs
  repo: https://imkt.uab.cat
- name: Python
  role: Used in the iMKTData pipeline notebooks to extract the per-gene unfolded SFS
    and divergence counts that feed the multi-gene analysis
  repo: https://github.com/jmurga/iMKTData
- name: Jupyter
  role: Hosts the step-by-step D. melanogaster and human pipeline notebooks used to
    build the input count tables
  repo: https://github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:c41f516d4a1a6b07126cbdfe0431a88ede3e8093a724f52e32f822c0cb45ab3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGD-UAB/iMKT
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bootstrap-confidence-interval-estimation

## Summary

Estimates a resampling 95% confidence interval for the fraction of adaptive evolution (alpha) in iMKT's multi-gene ('Concatenate genes') mode by re-estimating alpha over 100 bootstrap replicates that sample genes with replacement, alongside a Cochran-Mantel-Haenszel test for heterogeneity of selection among genes.

## When to use

Use when a gene set (e.g. selected by recombination rate bins or chromosome localization) needs a confidence interval on alpha rather than a single point estimate, and when homogeneity of selection across genes must be statistically assessed.

## When NOT to use

- The goal is a per-gene alpha estimate for individual genes rather than a gene-set-level regime and CI — in concatenate mode iMKT analyzes the whole gene set rather than each gene separately
- Input data are not polymorphism/divergence MKT counts (e.g. raw FASTA or VCF without the iMKTData pipeline's unfolded site frequency spectrum table having been produced first)

## Inputs

- Per-gene MKT divergence/polymorphism count tables (Dn, Ds, Pn, Ps) from D. melanogaster DGN/PopFly FASTA data
- User-defined gene set selection criterion (e.g. recombination rate bins or chromosome localization)
- Choice of MK-derived test (standard MKT, FWWMKT, eMKT, aMKT) with which alpha is estimated in each replicate

## Outputs

- Concatenated multi-gene MKT table of Dn, Ds, Pn and Ps counts
- Cochran-Mantel-Haenszel heterogeneity statistic for the gene set
- Bootstrap-derived resampling 95% CI for alpha (2.5th and 97.5th percentiles over 100 replicates), written to multi_gene_alpha_results.tsv

## How to apply

Load the per-gene MKT divergence/polymorphism count tables (Dn, Ds, Pn, Ps) derived from D. melanogaster DGN/PopFly FASTA data using the iMKT R package, and concatenate the user-selected gene set into a single multi-gene MKT table. Apply the Cochran-Mantel-Haenszel heterogeneity statistic across the per-gene 2x2 MKT contingency tables to test whether the selection regime is homogeneous over the gene set. Then run 100 bootstrap replicates, resampling genes with replacement, and re-estimate alpha in each replicate with the chosen MK-derived test (standard MKT, FWWMKT, eMKT or aMKT) via the iMKT pipeline. Compute the 2.5th and 97.5th percentiles of the bootstrap alpha distribution to obtain the resampling 95% confidence interval, and write the concatenated table, CMH statistic and 95% CI to the results file.

## Related tools

- **iMKT (R package)** (Performs the MK-derived tests to estimate alpha in each bootstrap replicate and provides the concatenate-mode analysis with CMH heterogeneity test and resampling CI) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web service offering the multi-gene 'Concatenate genes' mode for testing heterogeneity of selection and bootstrap CIs) — https://imkt.uab.cat
- **Python** (Used in the iMKTData pipeline notebooks to extract the per-gene unfolded SFS and divergence counts that feed the multi-gene analysis) — https://github.com/jmurga/iMKTData
- **Jupyter** (Hosts the step-by-step D. melanogaster and human pipeline notebooks used to build the input count tables) — https://github.com/jmurga/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- The bootstrap replicate count equals 100 and genes are sampled with replacement
- The output file multi_gene_alpha_results.tsv exists and contains both the CMH heterogeneity statistic and bootstrap-derived 95% CI fields
- The reported CI bounds correspond to the 2.5th and 97.5th percentiles of the bootstrap alpha distribution
- The concatenated multi-gene table contains the four MKT count columns (Dn, Ds, Pn, Ps) aggregated from the selected gene set
- Each bootstrap replicate's alpha is produced by the same MK-derived test chosen for the main analysis

## Limitations

- In concatenate mode the selective regime is estimated for the whole gene set rather than per gene, so per-gene alpha values are not produced
- The iMKT web service and R package behave differently across the four MK-derived tests (standard MKT, FWWMKT, eMKT, aMKT), so the CI depends on the chosen methodology
- Pipelines require downloaded external data (PopFly FASTA for D. melanogaster, PopHuman VCF for human) with paths that must be manually adjusted, and were run on a 100GB RAM / 16-core server

## Evidence

- [supplementary] concatenate mode CI and CMH: [evidence span withheld — anchor status: not_found]
- [intro] gene set comparison: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization)"
- [readme] package purpose: [evidence span withheld — anchor status: external_doc]
- [abstract] advanced options: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions"
