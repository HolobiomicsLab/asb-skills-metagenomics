---
name: mcdonald-kreitman-adaptation-estimation
description: Use when estimating adaptive evolution (alpha) from DAF and divergence
  tables for coding genes, or when comparing selective regimes of a user-selected
  gene set (e.g. recombination-rate bins) against a genome-wide distribution.
when_to_use_negative:
- Input is raw variant data (FASTA/VCF) rather than pre-computed DAF/divergence tables
  — run the iMKTData pipeline first to extract the unfolded site frequency spectrum
- Analyzing non-coding or non-protein-coding sequence — the data model is built on
  0-fold (selected) vs 4-fold (neutral) coding classes
- Human data lacking ancestral-state information or accessible-genome masking (e.g.
  the 1000GP pilot mask for low-quality variants) needed for correct polarisation
  and quality filtering
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0102
tools:
- name: iMKT (R package)
  role: Runs the standard MKT, eMKT, aMKT and FWWMKT on user-provided daf and divergence
    data frames to estimate alpha and site fractions
  repo: https://github.com/BGD-UAB/iMKT
- name: iMKT
  role: Web-based service (https://imkt.uab.cat) with advanced options such as 'Compare
    against whole-genome distribution' for testing gene sets against genome-wide selective
    regimes
  repo: https://imkt.uab.cat
- name: Python
  role: Used in the dmelProteins Jupyter notebook pipeline (python/bash) to build
    DAF/divergence tables from PopFly FASTA data
  repo: https://github.com/jmurga/iMKTData
- name: bash
  role: Raw scripts in the iMKTData src/ folder execute parallelizable pipeline steps
    under the imktData conda environment
  repo: https://github.com/jmurga/iMKTData
- name: Jupyter
  role: Step-by-step notebooks (dmelProteins.ipynb, humanProteins.ipynb, Python 3.6
    kernel) that execute the data-processing pipeline
  repo: https://github.com/jmurga/iMKTData
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz372
    title: 'iMKT: the integrative McDonald and Kreitman test'
  source_content_hash: sha256:d493734f9006af70941a5a7c3fa578826e264578f59fbe6d6bbc8b19c27c34bd
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
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mcdonald-kreitman-adaptation-estimation

## Summary

Estimate the fraction of adaptive substitutions (alpha) from polymorphism and divergence data using MK-derived tests (standard MKT, eMKT, aMKT, FWWMKT) implemented in the iMKT R package/web service, applied to D. melanogaster (DGN/PopFly) or human (1000GP/PopHuman) protein-coding genes.

## When to use

Use when estimating adaptive evolution (alpha) from DAF and divergence tables for coding genes, or when comparing selective regimes of a user-selected gene set (e.g. recombination-rate bins) against a genome-wide distribution.

## When NOT to use

- Input is raw variant data (FASTA/VCF) rather than pre-computed DAF/divergence tables — run the iMKTData pipeline first to extract the unfolded site frequency spectrum
- Analyzing non-coding or non-protein-coding sequence — the data model is built on 0-fold (selected) vs 4-fold (neutral) coding classes
- Human data lacking ancestral-state information or accessible-genome masking (e.g. the 1000GP pilot mask for low-quality variants) needed for correct polarisation and quality filtering

## Inputs

- Tab-delimited DAF table (derived allele frequency bins, Pi, P0 for 0-fold and 4-fold classes)
- Tab-delimited divergence table (mi, Di, m0, D0)
- D. melanogaster–D. simulans divergence data from PopFly in FASTA format (or DGN alignments from johnpool.net)
- Comeron et al. 2012 recombination map (cM/Mb per gene) for gene-set selection
- For human: 1000GP Phase III VCF with ancestral-state information from PopHuman and Bhèrer et al. 2017 recombination data

## Outputs

- Alpha estimate per gene or gene set for each MK-derived methodology (standard MKT, eMKT, aMKT, FWWMKT)
- Fractions of neutral (f), weakly deleterious (b) and strongly deleterious (d) sites
- MKT contingency table and Fisher's exact test P-value
- Divergence metrics: Ka, Ks, omega, omegaA, omegaD
- Results table (gene set, alpha estimates, mean/SD) e.g. emkt_alpha_results.tsv

## How to apply

Load unfolded site frequency spectrum (DAF) tables and divergence tables computed by the iMKTData pipelines from Drosophila Genome Nexus variation plus D. melanogaster–D. simulans divergence retrieved from PopFly in FASTA format (or 1000GP Phase III VCF plus human–chimpanzee divergence from PopHuman). Each table is tab-delimited, reporting 0-fold (selected) and 4-fold (neutral) functional classes with polymorphic (P, binned by derived allele frequency), divergent (D) and analyzed (m) sites. Run the eMKT through the iMKT R package (or the imkt.uab.cat web service) with the 'Compare against whole-genome distribution' option when testing a candidate gene set — e.g. the 475 Comeron et al. 2012 genes with recombination rate >7 cM/Mb — against all 13,753 RAL protein-coding genes. Compute mean and SD of alpha per gene set and compare against reference values (high-recombination mean alpha 0.602, SD 0.032; genome-wide mean alpha 0.44, SD 0.055) to validate the run. iMKT simultaneously returns alpha plus the fractions of neutral (f), weakly deleterious (b) and strongly deleterious (d) sites, so inspect these alongside alpha.

## Related tools

- **iMKT (R package)** (Runs the standard MKT, eMKT, aMKT and FWWMKT on user-provided daf and divergence data frames to estimate alpha and site fractions) — https://github.com/BGD-UAB/iMKT
- **iMKT** (Web-based service (https://imkt.uab.cat) with advanced options such as 'Compare against whole-genome distribution' for testing gene sets against genome-wide selective regimes) — https://imkt.uab.cat
- **Python** (Used in the dmelProteins Jupyter notebook pipeline (python/bash) to build DAF/divergence tables from PopFly FASTA data) — https://github.com/jmurga/iMKTData
- **bash** (Raw scripts in the iMKTData src/ folder execute parallelizable pipeline steps under the imktData conda environment) — https://github.com/jmurga/iMKTData
- **Jupyter** (Step-by-step notebooks (dmelProteins.ipynb, humanProteins.ipynb, Python 3.6 kernel) that execute the data-processing pipeline) — https://github.com/jmurga/iMKTData

## Examples

```
devtools::install_github("BGD-UAB/iMKT"); library(iMKT); standardMKT(myDafData, myDivergenceData)
```

## Evaluation signals

- High-recombination gene set (>7 cM/Mb, 475 genes) reproduces mean alpha = 0.602 (SD 0.032) for the D. melanogaster RAL population
- Whole-genome set (13,753 protein-coding genes) reproduces mean alpha = 0.44 (SD 0.055)
- Output includes a valid MKT table (neutral/selected class polymorphism and divergence counts) and a Fisher's exact test P-value (e.g. 1.48e-183 in the package example)
- Divergence metrics block reports Ka, Ks, omega, omegaA and omegaD; alpha symbol should fall in [0,1]
- Results table (emkt_alpha_results.tsv) contains one row per gene/gene set with alpha estimates matching the input gene-set membership

## Limitations

- iMKTData pipelines were developed in a specific conda environment (imktData.yml) on a 100GB RAM / 16-core server; file paths must be edited and memory/CPU constraints can limit local runs
- The iMKTData repository only contains raw code; users must construct custom bash scripts or manually parallelize multi-step processing
- MK-derived methodologies behave differently depending on data properties, hence iMKT provides all four simultaneously — a single test (e.g. standard MKT) can misestimate alpha under weakly deleterious polymorphism
- Results depend on correct ancestral-state polarisation (unfolded SFS) and on masking of low-quality variants (e.g. 1000GP pilot mask for human data)

## Evidence

- [abstract] Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?: "Advanced options in the website allow testing complex hypotheses such as the application example showed here: do genes located in high recombination regions undergo higher rates of adaptation?"
- [methods] Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly: "Variation data generated by the Drosophila Genome Nexus, together with divergence data between D. melanogaster and D. simulans, was retrieved from PopFly"
- [intro] It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the: "It allows comparing the selective regimes of a set of coding genes (selected according to the user's criterion, such as recombination rate bins or chromosome localization) with those of the"
- [readme] most functions require two input parameters: daf (data frame containing DAF, Pi and P0) and divergence (data frame containing mi, Di, m0, D0) and return the estimation of alpha: [evidence span withheld — anchor status: not_found]
- [readme] Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes: "Each pipeline returns a tab-delimited file including information unfolded site frequency spectrum, analyzable sites by largest transcrip and divergence by genes"
