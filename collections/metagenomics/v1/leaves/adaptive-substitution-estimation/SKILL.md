---
name: adaptive-substitution-estimation
description: Use when you have a DAF spectrum plus divergence counts (Ds, Dn, Ps,
  Pn or mi, Di, m0, D0) for selected (0-fold) and neutral (4-fold) classes and need
  to estimate α and classify the selection regime for coding genes.
when_to_use_negative:
- Data is a raw VCF or FASTA alignment without unfolded DAF/divergence counts — preprocess
  with the iMKTData pipelines first
- Derived allele frequency is unavailable (no ancestral-state information), since
  DAF-based exclusion of weakly deleterious variants and unfolded SFS computation
  cannot be performed
- The goal is per-gene selection classification rather than a single regime for a
  concatenated gene set — in concatenate mode iMKT analyzes the whole gene set, not
  each gene individually
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_2269
tools:
- name: iMKT
  role: Web-based service at https://imkt.uab.cat performing the four MKT types (standard
    MKT, FWWMKT, eMKT, aMKT) on user data or pre-loaded D. melanogaster and human
    protein-coding genes
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: R functions (e.g. standardMKT) that take daf and divergence data frames and
    return α, Fisher's exact test P-value, the MKT table and divergence metrics; supports
    CMH and bootstrap analyses for concatenated gene sets
  repo: https://github.com/BGD-UAB/iMKT
- name: Python
  role: Used in the Jupyter/bash iMKTData pipelines to build the tab-delimited DAF
    spectrum and divergence count tables that feed the MKT estimators
  repo: https://github.com/BGD-UAB/iMKTData
- name: Jupyter
  role: Notebooks (dmelProteins.ipynb, humanProteins.ipynb) execute the count-extraction
    pipeline step by step on a Python 3.6 kernel
  repo: https://github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:99cb609f99d484d013ea70217c50b18abf04e9920377dbd2500120760b2196f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# adaptive-substitution-estimation

## Summary

Estimates the fraction of adaptive substitutions (α) and other selection regimes (neutral, strongly deleterious, weakly deleterious) from polymorphism and divergence count tables using the four MKT variants implemented in iMKT (standard MKT, FWWMKT, eMKT, aMKT), optionally with the CMH heterogeneity test and bootstrap 95% CI for multi-gene sets.

## When to use

Use when you have a DAF spectrum plus divergence counts (Ds, Dn, Ps, Pn or mi, Di, m0, D0) for selected (0-fold) and neutral (4-fold) classes and need to estimate α and classify the selection regime for coding genes.

## When NOT to use

- Data is a raw VCF or FASTA alignment without unfolded DAF/divergence counts — preprocess with the iMKTData pipelines first
- Derived allele frequency is unavailable (no ancestral-state information), since DAF-based exclusion of weakly deleterious variants and unfolded SFS computation cannot be performed
- The goal is per-gene selection classification rather than a single regime for a concatenated gene set — in concatenate mode iMKT analyzes the whole gene set, not each gene individually

## Inputs

- daf data frame: derived allele frequency (DAF), Pi and P0 polymorphism counts per frequency bin (e.g. myDafData with bins 0.025, 0.075, 0.125, ...)
- divergence data frame: mi, Di, m0, D0 counts for selected and neutral classes (e.g. mi=2598805, Di=54641, m0=620019, D0=52537)
- Tab-delimited per-gene MKT table of Dn, Ds, Pn and Ps counts derived from DGN/PopFly FASTA or 1000GP/PopHuman VCF data
- User-specified DAF frequency cutoff(s) for excluding low-frequency (slightly deleterious) polymorphisms

## Outputs

- Estimate of α (fraction of adaptive substitutions) with the Fisher's exact test P-value
- MKT table of polymorphism and divergence counts per neutral and selected class
- Divergence metrics: Ka, Ks, ω, ωA, ωD
- CMH heterogeneity statistic and bootstrap-derived 95% CI for α (2.5th and 97.5th percentiles over 100 replicates)
- Estimates of the fraction of strongly deleterious (d), weakly deleterious (b) and neutral (f) sites

## How to apply

Load a daf data frame containing DAF, Pi and P0 columns and a divergence data frame containing mi, Di, m0, D0 (tab-delimited, as produced by the iMKTData pipelines). Choose the MKT variant: standard MKT computes α = 1 − (Ds/Dn)/(Ps/Pn) directly, while eMKT, FWWMKT and aMKT first exclude weakly deleterious low-frequency polymorphisms by applying a user-specified derived allele frequency cutoff to the DAF spectrum and recomputing corrected Ps and Pn before estimating α. For a multi-gene set (e.g. recombination rate bins or chromosome localization), concatenate the per-gene 2x2 MK contingency tables, apply the Cochran-Mantel-Haenszel statistic to test homogeneity of selection across genes, and run 100 bootstrap replicates sampling genes with replacement, estimating α in each replicate with the chosen MKT test. Compute the 2.5th and 97.5th percentiles of the bootstrap α distribution as the resampling 95% CI. Report α together with divergence metrics (Ka, Ks, ω, ωA, ωD) and the Fisher's exact test P-value.

## Related tools

- **iMKT** (Web-based service at https://imkt.uab.cat performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT) on user data or pre-loaded D. melanogaster and human protein-coding genes) — https://imkt.uab.cat
- **iMKT (R package)** (R functions (e.g. standardMKT) that take daf and divergence data frames and return α, Fisher's exact test P-value, the MKT table and divergence metrics; supports CMH and bootstrap analyses for concatenated gene sets) — https://github.com/BGD-UAB/iMKT
- **Python** (Used in the Jupyter/bash iMKTData pipelines to build the tab-delimited DAF spectrum and divergence count tables that feed the MKT estimators) — https://github.com/BGD-UAB/iMKTData
- **Jupyter** (Notebooks (dmelProteins.ipynb, humanProteins.ipynb) execute the count-extraction pipeline step by step on a Python 3.6 kernel) — https://github.com/jmurga/iMKTData

## Examples

```
standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- The MKT table reports exactly two classes (Neutral class and Selected class) with polymorphism and divergence counts matching the input daf/divergence frames
- Recomputing α = 1 − (Ds/Dn)/(Ps/Pn) from the stored Ds, Dn, Ps, Pn values reproduces the reported α estimate (e.g. 0.2364499 for the myDafData/myDivergenceData example)
- A Fisher's exact test P-value is reported and is highly significant for the reference example (1.480943e-183)
- For concatenated analyses, the bootstrap replicate count equals 100 and the CI is defined by the 2.5th and 97.5th percentiles of the bootstrap α distribution
- Divergence metrics block contains Ka, Ks, ω, ωA and ωD (reference example: Ka=0.0210254, Ks=0.0847345, ω=0.2481331)

## Limitations

- eMKT, FWWMKT and aMKT require the user to specify one or several DAF threshold frequency values to exclude weakly deleterious low-frequency polymorphisms; results are cutoff-dependent
- In concatenate mode, selective regimes are inferred for the whole gene set rather than per gene, so heterogeneous regimes within a set must be flagged via the CMH statistic
- The iMKTData pipelines require large resources (100GB RAM, 16 Xeon cores) and paths must be manually adjusted; pipelines were run in the imktData.yml conda environment
- The R package must be installed from GitHub via devtools::install_github("BGD-UAB/iMKT")

## Evidence

- [abstract] four MKT types at once: "Here we present iMKT (acronym for integrative McDonald and Kreitman test), a novel web-based service performing four distinct MKT types."
- [abstract] four selection regimes: "It allows the detection and estimation of four different selection regimes −adaptive, neutral, strongly deleterious and weakly deleterious− acting on any genomic sequence."
- [readme] five MK-derived methods for alpha: [evidence span withheld — anchor status: external_doc]
- [readme] input requirements: [evidence span withheld — anchor status: external_doc]
- [supplementary] eMKT low-frequency exclusion: [evidence span withheld — anchor status: not_found]
- [methods] CMH and bootstrap CI: [evidence span withheld — anchor status: not_found]
