---
name: population-genomics-mkt-estimation
description: Use when population-genomic polymorphism (DAF spectra for 0-fold and
  4-fold classes) and divergence counts between D. melanogaster and D. simulans (or
  human-chimp) are available and you must estimate alpha, e.g. comparing gene sets
  such as high-recombination regions against the whole genome.
when_to_use_negative:
- You lack unfolded polymorphism data with divergence between an outgroup (e.g. D.
  simulans) — MK-derived tests require polymorphic, divergent and analyzed site counts
  for both 0-fold and 4-fold classes
- You need de novo variant calling or population structure inference rather than alpha
  estimation from pre-computed SFS/divergence tables
- Your input is human data without ancestral-state information from 1000GP Phase III,
  which is needed to unfold the SFS
edam_topics:
- http://edamontology.org/topic_3056
- http://edamontology.org/topic_0199
tools:
- name: iMKT
  role: Web service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT)
    on pre-loaded D. melanogaster and human protein-coding gene data or user data
  repo: https://imkt.uab.cat
- name: iMKT (R package)
  role: R functions (standardMKT, FWWMKT, eMKT, aMKT) that take daf and divergence
    data frames and return alpha, Fisher's exact test P-value and divergence metrics
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKTData pipelines
  role: Python/bash/Jupyter pipelines to extract unfolded site frequency spectrum
    and divergence-by-gene tables from DGN FASTA and 1000GP VCF for use with iMKT
  repo: https://github.com/BGD-UAB/iMKTData
- name: PopFly
  role: Source of pre-loaded D. melanogaster population genomic data (e.g. ZI population)
    and divergence for iMKT analyses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:f359be8eb3d13a3ec4f4330ddb7afb31e5fbf9f67b2cc7a070caaccb2fd817a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PopFly
  - Python
  - bash
  - iMKT
  - iMKT (R package)
  - iMKTData pipelines
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3056
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# population-genomics-mkt-estimation

## Summary

Estimates the fraction of adaptive evolution (alpha) and other selection regimes (strongly deleterious d, weakly deleterious b, neutral f) from unfolded polymorphism and divergence site counts using MK-derived tests (standard MKT, FWW, eMKT, aMKT) implemented in the iMKT R package / web service, with pre-loaded D. melanogaster (PopFly/DGN) and human (PopHuman/1000GP) data.

## When to use

Use when population-genomic polymorphism (DAF spectra for 0-fold and 4-fold classes) and divergence counts between D. melanogaster and D. simulans (or human-chimp) are available and you must estimate alpha, e.g. comparing gene sets such as high-recombination regions against the whole genome.

## When NOT to use

- You lack unfolded polymorphism data with divergence between an outgroup (e.g. D. simulans) — MK-derived tests require polymorphic, divergent and analyzed site counts for both 0-fold and 4-fold classes
- You need de novo variant calling or population structure inference rather than alpha estimation from pre-computed SFS/divergence tables
- Your input is human data without ancestral-state information from 1000GP Phase III, which is needed to unfold the SFS

## Inputs

- Tab-delimited per-gene polymorphism table with unfolded site frequency spectrum (DAF, Pi, P0) split by MKT functional class (0-fold selected, 4-fold neutral)
- Tab-delimited per-gene divergence table (mi, Di, m0, D0) between D. melanogaster and D. simulans, or human-chimp from PopHuman
- Gene partition of interest (e.g. high-recombination gene set defined by the Comeron et al. 2012 recombination map) plus the full genome-wide gene set
- D. melanogaster variation data from the Drosophila Genome Nexus / PopFly (FASTA) or 1000GP Phase III VCF for human

## Outputs

- Per gene-set estimates of alpha with Fisher's exact test P-value and MK contingency table (neutral vs selected class polymorphism/divergence)
- Divergence metrics: Ka, Ks, omega, omegaA, omegaD
- Fractions of strongly deleterious (d), weakly deleterious (b) and neutral (f) sites
- Group-level summary statistics (mean and SD of alpha) for the target gene set vs genome-wide, e.g. written to a TSV such as results_zi_emkt_recombination.tsv

## How to apply

Obtain the per-gene tab-delimited table of unfolded site frequency spectrum, analyzable sites and divergence by functional class (0-fold neutral / 4-fold selected) via the iMKTData pipelines or PopFly/PopHuman pre-loaded data. Partition genes by a user criterion (e.g. the Comeron et al. 2012 recombination map for high-recombination bins). Format two inputs per analysis: a daf data frame (columns daf, Pi, P0) and a divergence data frame (columns mi, Di, m0, D0), then call the appropriate iMKT function — standardMKT, FWWMKT, eMKT (excludes weakly deleterious variants) or aMKT — to estimate alpha. Compare summary statistics (mean and SD of alpha) between the selected gene set and the genome-wide distribution to test the hypothesis. Rationale: the eMKT specifically corrects for weakly deleterious polymorphisms that inflate Pi and bias alpha downward.

## Related tools

- **iMKT** (Web service performing the four MKT types (standard MKT, FWWMKT, eMKT, aMKT) on pre-loaded D. melanogaster and human protein-coding gene data or user data) — https://imkt.uab.cat
- **iMKT (R package)** (R functions (standardMKT, FWWMKT, eMKT, aMKT) that take daf and divergence data frames and return alpha, Fisher's exact test P-value and divergence metrics) — https://github.com/BGD-UAB/iMKT
- **iMKTData pipelines** (Python/bash/Jupyter pipelines to extract unfolded site frequency spectrum and divergence-by-gene tables from DGN FASTA and 1000GP VCF for use with iMKT) — https://github.com/BGD-UAB/iMKTData
- **PopFly** (Source of pre-loaded D. melanogaster population genomic data (e.g. ZI population) and divergence for iMKT analyses)

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- Computed mean alpha for the high-recombination gene set reproduces the reported 0.633 (SD 0.028) and genome-wide alpha 0.457 (SD 0.053) for the 13,753 D. melanogaster ZI genes
- Gene set totals match the expected 13,753 protein-coding D. melanogaster genes when partitioned by the Comeron et al. 2012 recombination map
- Each run returns a valid MK contingency table (neutral/selected class polymorphism and divergence counts) and a Fisher's exact test P-value
- Divergence metrics (Ka, Ks, omega, omegaA, omegaD) are finite and consistent with the input counts (e.g. omega = 0.2481 for the package sample data)
- Output TSV contains per-group alpha mean and SD matching the schema of results_zi_emkt_recombination.tsv

## Limitations

- Standard MKT alpha is biased by weakly deleterious polymorphisms; use eMKT/aMKT/FWW variants when weakly deleterious variants are present (iMKT detects four selection regimes: adaptive, neutral, strongly deleterious, weakly deleterious)
- iMKTData pipelines were developed in a conda environment (imktData.yml) on a local server with 100GB RAM and 16 Intel Xeon CPUs; paths in the pipelines must be manually changed and multi-step processing must be parallelized by custom bash scripts
- Only raw code is provided in iMKTData; reproducing results requires downloading external files (PopFly FASTA, Comeron et al. 2012 recombination data, 1000GP Phase III VCF and pilot mask)

## Evidence

- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [supplementary] The corresponding R package is available for download at https://github.com/BGD-UAB/iMKT: "The corresponding R package is available for download at https://github.com/BGD-UAB/iMKT"
- [readme] It includes five MK derived methodologies which allow inferring the rate of adaptive evolution (α) as well as the fraction of strongly deleterious (d), weakly deleterious (b), and neutral (f) sites.: [evidence span withheld — anchor status: external_doc]
- [readme] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
