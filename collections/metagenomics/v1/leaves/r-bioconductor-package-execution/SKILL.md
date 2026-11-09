---
name: r-bioconductor-package-execution
description: Use when a taxa-by-sample abundance table must be converted into a square
  taxa–taxa correlation (R) matrix with per-pair R² goodness-of-fit values as preprocessing
  for mbQTL rho estimation.
when_to_use_negative:
- The goal is taxa–SNP rho estimation itself and the taxa–taxa R matrix has already
  been computed via coringTaxa
- The input is a phyloseq S4 object or metagenomeSeq MRexperiment object that still
  needs conversion via phyloseq_to_metagenomeSeq()/metagenomeSeqToMbqtl before it
  is an mbQTL-compatible data frame
- The analysis targets logistic regression of presence/absence (categorical) taxa
  against SNPs rather than correlation estimation
edam_operation: http://edamontology.org/operation_2238
tools:
- name: mbQTL
  role: R/Bioconductor package whose coringTaxa() function computes the taxa–taxa
    correlation (R) matrix and R² goodness-of-fit measures consumed by Approach B
    rho estimation
  repo: https://github.com/Mercedeh66/mbQTL
- name: R
  role: Execution environment in which mbQTL is installed from Bioconductor/GitHub
    and the vignette workflow (data("microbeAbund"); coringTaxa) is run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:ef88eda56e4192927195257ba19d035e5541fe624b596528c50501d13f47625e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mercedeh66/mbQTL
  tools:
  - R
  - mbQTL
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-bioconductor-package-execution

## Summary

Executes the mbQTL Bioconductor package in R to reproduce the taxa–taxa correlation (R) matrix and goodness-of-fit (R²) measures that feed mbQTL's Approach B rho estimation for taxa–SNP associations.

## When to use

Use when a taxa-by-sample abundance table must be converted into a square taxa–taxa correlation (R) matrix with per-pair R² goodness-of-fit values as preprocessing for mbQTL rho estimation.

## When NOT to use

- The goal is taxa–SNP rho estimation itself and the taxa–taxa R matrix has already been computed via coringTaxa
- The input is a phyloseq S4 object or metagenomeSeq MRexperiment object that still needs conversion via phyloseq_to_metagenomeSeq()/metagenomeSeqToMbqtl before it is an mbQTL-compatible data frame
- The analysis targets logistic regression of presence/absence (categorical) taxa against SNPs rather than correlation estimation

## Inputs

- taxa-by-sample microbial abundance data frame (e.g. microbeAbund; rownames taxa, colnames sample names)
- optionally a metagenomeSeq MRexperiment object or phyloseq S4 object converted via phyloseq_to_metagenomeSeq() and metagenomeSeqToMbqtl

## Outputs

- square taxa-by-taxa correlation matrix (correlationMicrobes, R values) with off-diagonal entries in [-1, 1]
- one R² goodness-of-fit entry per taxon pair, suitable for combination with SNP–taxa R² via taxaSnpCor (e.g. exported as coringTaxa_R_table.csv)

## How to apply

Load mbQTL from Bioconductor (BiocManager::install("mbQTL")) or GitHub (devtools::install_github('Mercedeh66/mbQTL')) and attach the taxa abundance data frame with data("microbeAbund"), where rownames are taxa and colnames are sample names. Call coringTaxa(microbeAbund) to compute the taxa–taxa correlation matrix (R values) and associated R² goodness-of-fit measures. Write the returned correlation table (e.g. coringTaxa_R_table.csv) and confirm it is a square taxa-by-taxa table with off-diagonal values in [-1, 1]. This R matrix is required because mbQTL estimates rho values for taxa–SNP correlation after the taxa–taxa R values are estimated, then adjusted for the association of taxa–SNP, so the table must be combined with SNP–taxa R² via taxaSnpCor before rho estimation.

## Related tools

- **mbQTL** (R/Bioconductor package whose coringTaxa() function computes the taxa–taxa correlation (R) matrix and R² goodness-of-fit measures consumed by Approach B rho estimation) — https://github.com/Mercedeh66/mbQTL
- **R** (Execution environment in which mbQTL is installed from Bioconductor/GitHub and the vignette workflow (data("microbeAbund"); coringTaxa) is run)

## Examples

```
library(mbQTL); data("microbeAbund"); correlationMicrobes <- coringTaxa(microbeAbund)
```

## Evaluation signals

- Output is a square taxa-by-taxa table matching the dimensions of the input abundance data frame
- All off-diagonal correlation (R) values lie within [-1, 1]
- One R² goodness-of-fit entry exists per taxon pair
- The table's structure matches what taxaSnpCor expects when combining taxa–taxa R with SNP–taxa R² before rho estimation
- The vignette-style call data("microbeAbund"); coringTaxa(microbeAbund) reproduces the correlationMicrobes object without errors

## Limitations

- The vignette datasets (Taxa_table_MVP_cor.txt, SNP_table_MVP_cor.txt, microbeAbund) are simulated and not factual, so reproducing them validates mechanics, not biological conclusions
- Inputs must already be mbQTL-compatible data frames; metagenomeSeq or phyloseq objects require normalization/conversion steps first
- The R matrix alone does not yield taxa–SNP associations; it must be combined with SNP–taxa R² measures via taxaSnpCor for Approach B

## Evidence

- [intro] Estimate rho values for taxa-SNP correlation after estimating taxa-taxa R values: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] Taxa abundance input format: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [readme] Bioconductor installation command: [evidence span withheld — anchor status: external_doc]
