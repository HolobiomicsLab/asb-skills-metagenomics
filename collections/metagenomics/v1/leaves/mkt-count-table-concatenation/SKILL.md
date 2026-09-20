---
name: mkt-count-table-concatenation
description: Use when a user-selected set of protein-coding genes (e.g. selected by
  recombination rate bins or chromosome localization) must be analyzed jointly for
  a single selective regime rather than gene-by-gene, and per-gene MKT count tables
  are available.
when_to_use_negative:
- The goal is gene-by-gene selective regime estimates rather than a regime for the
  whole gene set — concatenate mode analyzes the set jointly, not individual genes
- The per-gene MKT count tables (Dn, Ds, Pn, Ps) have not yet been extracted; run
  the iMKTData pipeline (e.g. dmelProteins.ipynb) first
- The data are human (1000GP/PopHuman VCF) and the CMH/bootstrap concatenated workflow
  has only been documented for the D. melanogaster DGN/PopFly protein-coding example
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: iMKT
  role: Web service and R package providing the 'Concatenate genes' multi-gene analysis
    mode, the MK tests used to estimate alpha in each bootstrap replicate, and the
    CMH heterogeneity test
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT (R package)
  role: R functions (e.g. standardMKT and related MK-derived methodologies) that consume
    daf and divergence data frames to estimate alpha for bootstrap resampling of genes
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Language of the D. melanogaster pipeline (dmelProteins.ipynb) that generates
    the per-gene unfolded SFS and divergence count tables fed into the concatenated
    analysis
  repo: https://github.com/jmurga/iMKTData
- name: bash
  role: Raw scripts in the iMKTData src/ folder used to execute the count-table extraction
    pipeline steps
  repo: https://github.com/jmurga/iMKTData
- name: Jupyter
  role: Notebooks (Python 3.6 kernel) that run the D. melanogaster pipeline step by
    step to produce the gene-level MKT tables
  repo: https://github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:4ab0b9457f1f40f29d1362a5257c9e09188ed71769b228b2fc78876bf53219bb
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mkt-count-table-concatenation

## Summary

Concatenates per-gene MKT divergence/polymorphism count tables (Dn, Ds, Pn, Ps) into a single multi-gene contingency table and statistically assesses heterogeneity of selection among genes using the Cochran-Mantel-Haenszel test plus a bootstrap 95% CI for alpha, as implemented in iMKT's 'Concatenate genes' mode.

## When to use

Use when a user-selected set of protein-coding genes (e.g. selected by recombination rate bins or chromosome localization) must be analyzed jointly for a single selective regime rather than gene-by-gene, and per-gene MKT count tables are available.

## When NOT to use

- The goal is gene-by-gene selective regime estimates rather than a regime for the whole gene set — concatenate mode analyzes the set jointly, not individual genes
- The per-gene MKT count tables (Dn, Ds, Pn, Ps) have not yet been extracted; run the iMKTData pipeline (e.g. dmelProteins.ipynb) first
- The data are human (1000GP/PopHuman VCF) and the CMH/bootstrap concatenated workflow has only been documented for the D. melanogaster DGN/PopFly protein-coding example

## Inputs

- Per-gene MKT divergence/polymorphism count tables (Dn, Ds, Pn, Ps) derived from D. melanogaster DGN/PopFly FASTA data
- User-selected gene set (e.g. defined by recombination rate bins from Comeron et al. 2012 or chromosome localization)
- DAF data frame (daf: DAF, Pi, P0) and divergence data frame (mi, Di, m0, D0) as required by iMKT R package functions

## Outputs

- Concatenated multi-gene MKT table of Dn, Ds, Pn and Ps counts
- Cochran-Mantel-Haenszel heterogeneity statistic for the gene set
- Resampling 95% confidence interval for alpha (2.5th and 97.5th percentiles over 100 bootstrap replicates), written to multi_gene_alpha_results.tsv

## How to apply

Load per-gene MKT divergence/polymorphism count tables derived from D. melanogaster DGN/PopFly FASTA data using the iMKT R package, then concatenate the gene set selected by the user's criterion (e.g. recombination rate bins or chromosome localization) into one multi-gene table of Dn, Ds, Pn and Ps counts. Apply the Cochran-Mantel-Haenszel heterogeneity statistic across the per-gene 2x2 MKT contingency tables to test whether the selection regime is homogeneous over the gene set. Then run 100 bootstrap replicates resampling genes with replacement, estimating alpha in each replicate with the chosen iMKT MK test (standard MKT, FWWMKT, eMKT or aMKT), and take the 2.5th and 97.5th percentiles of the bootstrap alpha distribution as the resampling 95% confidence interval. Rationale: concatenation pools information for the whole gene set, while the CMH test guards against falsely assuming a uniform selective regime across genes.

## Related tools

- **iMKT** (Web service and R package providing the 'Concatenate genes' multi-gene analysis mode, the MK tests used to estimate alpha in each bootstrap replicate, and the CMH heterogeneity test) — https://github.com/BGD-UAB/iMKT
- **iMKT (R package)** (R functions (e.g. standardMKT and related MK-derived methodologies) that consume daf and divergence data frames to estimate alpha for bootstrap resampling of genes) — https://github.com/BGD-UAB/iMKT
- **Python** (Language of the D. melanogaster pipeline (dmelProteins.ipynb) that generates the per-gene unfolded SFS and divergence count tables fed into the concatenated analysis) — https://github.com/jmurga/iMKTData
- **bash** (Raw scripts in the iMKTData src/ folder used to execute the count-table extraction pipeline steps) — https://github.com/jmurga/iMKTData
- **Jupyter** (Notebooks (Python 3.6 kernel) that run the D. melanogaster pipeline step by step to produce the gene-level MKT tables) — https://github.com/jmurga/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- multi_gene_alpha_results.tsv exists and contains both the CMH heterogeneity statistic and the bootstrap-derived 95% CI fields
- The bootstrap replicate count equals 100, with genes sampled with replacement
- The 95% CI bounds correspond to the 2.5th and 97.5th percentiles of the bootstrap alpha distribution
- The concatenated table contains exactly the pooled Dn, Ds, Pn and Ps counts for the user-selected gene set
- Per-gene 2x2 MKT contingency tables feed the CMH test and match the input count tables

## Limitations

- In concatenate mode the selective regime is estimated for the whole gene set, not per gene, so individual genes contributing to heterogeneity are not resolved (the CMH test only flags heterogeneity)
- The CMH result depends on the assumption that per-gene 2x2 tables are comparable; strong heterogeneity undermines a single pooled alpha
- The bootstrap CI is a resampling (gene-level) interval over 100 replicates and does not capture other sources of uncertainty
- The iMKT R package workflow presumes inputs formatted as daf and divergence data frames; paths in the iMKTData pipelines must be adapted to the local server

## Evidence

- [other] In concatenate mode, iMKT analyzes selective regimes for the whole gene set rather than per gene, applies the Cochran-Mantel-Haenszel statistic: [evidence span withheld — anchor status: not_found]
- [other] generates a resampling 95% CI for alpha by estimating it with the chosen MKT test over 100 bootstrap replicates sampling genes with replacement: [evidence span withheld — anchor status: not_found]
- [intro] It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization): "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization)"
- [readme] most functions require two input parameters: daf (data frame containing DAF, Pi and P0) and divergence (data frame containing mi, Di, m0, D0): [evidence span withheld — anchor status: not_found]
- [methods] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
