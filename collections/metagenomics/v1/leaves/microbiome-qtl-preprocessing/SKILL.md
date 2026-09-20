---
name: microbiome-qtl-preprocessing
description: Use when a taxa-by-sample 16S abundance table (plus an optional SNP/covariate
  file) must be prepared for mbQTL rho estimation, and taxa–taxa R values need to
  be estimated as inputs to taxa–SNP association analysis.
when_to_use_negative:
- The analysis goal is mbQTL's Approach A linear regression (MatrixEQTL model) or
  the logistic regression for presence/absence (categorical) taxa, which do not require
  the coringTaxa taxa–taxa R matrix as preprocessing.
- The input is already a taxa–taxa correlation matrix or a pre-estimated R² goodness-of-fit
  table, in which case you can proceed directly to rho estimation/taxaSnpCor.
- The data are not 16S rRNA abundance plus SNP/SNV data — mbQTL is designed for integrating
  16S rRNA sequencing with SNV/SNP data.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: mbQTL
  role: R/Bioconductor package whose coringTaxa() function computes the taxa–taxa
    correlation matrix (R values) and R² measures feeding the rho estimation step
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: runtime environment in which mbQTL and its example data (microbeAbund, SnpFile,
    CovFile) are loaded and executed
- name: phyloseq
  role: provides phyloseq_to_metagenomeSeq() to convert phyloseq S4 objects into metagenomeSeq
    format before mbQTL preprocessing
- name: metagenomeSeq
  role: MRexperiment objects can be normalized or aggregated to taxonomic levels and
    converted to mbQTL-compatible data frames prior to correlation estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:3dd138a84092dafa7a15f41924445b94774a67225d08d6c649afa835d66aec1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mbQTL
  - metagenomeSeq
  - phyloseq
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# microbiome-qtl-preprocessing

## Summary

Preprocess microbiome and genotype inputs for mbQTL mbQTL estimation: load the taxa-by-sample abundance data (microbeAbund) and compute the taxa–taxa correlation matrix (R values) and R² goodness-of-fit measures with coringTaxa, which are required before the Approach B rho estimation of taxa–SNP correlations.

## When to use

Use when a taxa-by-sample 16S abundance table (plus an optional SNP/covariate file) must be prepared for mbQTL rho estimation, and taxa–taxa R values need to be estimated as inputs to taxa–SNP association analysis.

## When NOT to use

- The analysis goal is mbQTL's Approach A linear regression (MatrixEQTL model) or the logistic regression for presence/absence (categorical) taxa, which do not require the coringTaxa taxa–taxa R matrix as preprocessing.
- The input is already a taxa–taxa correlation matrix or a pre-estimated R² goodness-of-fit table, in which case you can proceed directly to rho estimation/taxaSnpCor.
- The data are not 16S rRNA abundance plus SNP/SNV data — mbQTL is designed for integrating 16S rRNA sequencing with SNV/SNP data.

## Inputs

- Taxa-by-sample microbial abundance data frame (microbeAbund; rownames taxa, colnames sample names)
- SnpFile: SNP/genotype matrix (rownames snps, colnames sample names)
- Optional CovFile of covariates (rownames covariates, colnames sample names)
- metagenomeSeq MRexperiment object or phyloseq S4 object as alternative starting formats

## Outputs

- correlationMicrobes: square taxa-by-taxa correlation table (R values)
- R² goodness-of-fit entry per taxon pair, required before mbQTL's rho estimation step

## How to apply

Load the mbQTL package in R and attach the packaged example data with data("microbeAbund"), a taxa-by-sample abundance data frame where rownames are taxa and colnames are sample names; optional companion inputs are SnpFile (rownames snps, colnames samples) and CovFile. If starting from an MRexperiment object, normalize or aggregate it with mbQTL's metagenomeSeq conversion function; phyloseq S4 objects must first be converted with phyloseq_to_metagenomeSeq(). Call coringTaxa(microbeAbund) to compute the taxa–taxa correlation matrix (R values) together with the goodness-of-fit (R²) measures. The rationale is that mbQTL's rho estimation is 'a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP', so this correlation preprocessing is a prerequisite for the Approach B rho step that simultaneously estimates taxa–SNP associations and identifies linkage disequilibrium (LD) SNPs and regions.

## Related tools

- **mbQTL** (R/Bioconductor package whose coringTaxa() function computes the taxa–taxa correlation matrix (R values) and R² measures feeding the rho estimation step) — https://bioconductor.org/packages/mbQTL/
- **R** (runtime environment in which mbQTL and its example data (microbeAbund, SnpFile, CovFile) are loaded and executed)
- **phyloseq** (provides phyloseq_to_metagenomeSeq() to convert phyloseq S4 objects into metagenomeSeq format before mbQTL preprocessing)
- **metagenomeSeq** (MRexperiment objects can be normalized or aggregated to taxonomic levels and converted to mbQTL-compatible data frames prior to correlation estimation)

## Examples

```
data("microbeAbund"); correlationMicrobes <- coringTaxa(microbeAbund)
```

## Evaluation signals

- Output of coringTaxa is a square taxa-by-taxa table (same taxa on rows and columns).
- Off-diagonal correlation values (R) all lie within [-1, 1].
- There is one R² (goodness-of-fit) entry per taxon pair, matching the structure required before mbQTL's rho estimation step.
- Input abundance frame conforms to mbQTL conventions: rownames are taxa, colnames are sample names (optionally with SnpFile/CovFile following the same sample ordering).
- Downstream combination with SNP–taxa R² via taxaSnpCor succeeds, indicating the R matrix is in the expected format.

## Limitations

- The vignette datasets (microbeAbund, SnpFile, CovFile) are simulated and are not factual, so example outputs should not be interpreted as biological results.
- covariates.txt-style CovFile input applies to the linear regression section, not necessarily to all mbQTL approaches.
- Taxa–SNP relationships are estimated after taxa–taxa R estimation; skipping this preprocessing breaks the rho estimation workflow for Approach B.

## Evidence

- [intro] rho estimation is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP: "rho estimation for correlation of taxa and SNP, which is a novel method of estimating rho values after R value of taxa-taxa is estimated then adjusted for association of taxa-SNP"
- [intro] mbQTL input file conventions for the abundance data: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [intro] phyloseq objects must be converted before mbQTL preprocessing: "they can use the `phyloseq_to_metagenomeSeq()` option of the `phyloseq` package and start from there."
- [readme] Installation of mbQTL from GitHub for preprocessing runs: [evidence span withheld — anchor status: external_doc]
