---
name: taxa-snp-linear-regression-qtl-mapping
description: Use when you have a microbial abundance matrix (taxa x samples), a genotype/SNP
  matrix (SNPs x samples), optional covariates, and you need to detect significant
  taxa–SNP host-genetic associations via linear regression.
when_to_use_negative:
- Taxa are presence/absence (categorical) rather than continuous abundances — mbQTL
  provides logistic regression for that case instead
- The goal is correlation-based rho estimation or identification of linkage disequilibrium
  (LD) SNPs via taxa-taxa R values (Approach B), rather than per-pair linear regression
- Input is not in the required sample-matched taxa/SNP data-frame format and has not
  been converted via metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq()
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: mbQTL
  role: R/Bioconductor package providing linearTaxaSnp(), histPvalueLm(), and qqPlotLm()
    for taxa–SNP linear regression QTL mapping
  repo: https://bioconductor.org/packages/mbQTL/
- name: R
  role: Runtime environment in which mbQTL is installed and run
- name: MatrixEQTL
  role: Core linear regression model underlying linearTaxaSnp()
- name: phyloseq
  role: 'Optional preprocessing: phyloseq_to_metagenomeSeq() converts phyloseq S4
    objects to metagenomeSeq format before mbQTL analysis'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:14f39bcc2fcf5c3f977a62c338e8dae11a63477285e9f782a9453bc989a393bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MatrixEQTL
  - R
  - mbQTL
  - phyloseq
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# taxa-snp-linear-regression-qtl-mapping

## Summary

mbQTL's Approach A identifies microbial QTLs by covariate-adjusted linear regression of each taxon's abundance against each host SNP using the core MatrixEQTL model, with P value diagnostics (histPvalueLm, qqPlotLm) and FDR/Bonferroni multiple-testing reporting.

## When to use

Use when you have a microbial abundance matrix (taxa x samples), a genotype/SNP matrix (SNPs x samples), optional covariates, and you need to detect significant taxa–SNP host-genetic associations via linear regression.

## When NOT to use

- Taxa are presence/absence (categorical) rather than continuous abundances — mbQTL provides logistic regression for that case instead
- The goal is correlation-based rho estimation or identification of linkage disequilibrium (LD) SNPs via taxa-taxa R values (Approach B), rather than per-pair linear regression
- Input is not in the required sample-matched taxa/SNP data-frame format and has not been converted via metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq()

## Inputs

- microbeAbund: microbial abundance data frame (rownames taxa, colnames sample names)
- SnpFile: SNP/genotype matrix (rownames SNPs, colnames sample names)
- CovFile (optional): covariate matrix (rownames covariates, colnames sample names)
- Optionally a metagenomeSeq MRexperiment or phyloseq S4 object to convert first

## Outputs

- Linear regression output object with per taxon–SNP pair statistics (coefficient, P value, FDR-adjusted P value)
- FDR- and Bonferroni-adjusted association statistics table (e.g. linear_taxa_snp_results.tsv)
- P value diagnostic plots from histPvalueLm() and qqPlotLm()

## How to apply

Load mbQTL's example-format inputs with data('microbeAbund'), data('SnpFile'), and data('CovFile'); microbeAbund holds taxa (rownames) by samples (colnames), SnpFile holds SNPs by samples, and CovFile optionally holds covariates by samples. Call linearTaxaSnp(microbeAbund, SnpFile, Covariate = CovFile), which runs covariate-adjusted linear regression of every taxon against every SNP on the core MatrixEQTL model. Extract per taxon–SNP pair statistics — coefficient, P value, and FDR-adjusted P value — from the returned linear regression output object, and report FDR and Bonferroni-adjusted results. Assess P value validity using the accompanying histPvalueLm() (P value histogram, expected uniform under the null) and qqPlotLm() (QQ plot) diagnostics. If your microbiome data is a metagenomeSeq MRexperiment or phyloseq object, first normalize/aggregate it via metagenomeSeqToMbqtl or convert with phyloseq_to_metagenomeSeq() to reach the required data-frame format.

## Related tools

- **mbQTL** (R/Bioconductor package providing linearTaxaSnp(), histPvalueLm(), and qqPlotLm() for taxa–SNP linear regression QTL mapping) — https://bioconductor.org/packages/mbQTL/
- **R** (Runtime environment in which mbQTL is installed and run)
- **MatrixEQTL** (Core linear regression model underlying linearTaxaSnp())
- **phyloseq** (Optional preprocessing: phyloseq_to_metagenomeSeq() converts phyloseq S4 objects to metagenomeSeq format before mbQTL analysis)

## Examples

```
linearTaxaSnp(microbeAbund, SnpFile, Covariate = CovFile)
```

## Evaluation signals

- The output file exists and contains rows for taxon–SNP pairs with both P value and FDR columns populated
- P value histogram from histPvalueLm() is approximately uniform (no inflation artifact)
- QQ plot from qqPlotLm() shows expected null distribution with only a tail of associations deviating
- FDR and Bonferroni adjustments are present alongside raw P values in the results table
- Sample names align across microbeAbund, SnpFile, and CovFile (colnames match) before regression

## Limitations

- Linear regression treats taxa as continuous abundances; it is not the appropriate mbQTL approach for presence/absence (categorical) taxa, where logistic regression is used
- Approach A does not itself identify linkage disequilibrium (LD) SNPs/regions; the article's rho-estimation method adjusts for taxa-taxa R values and is a separate analysis
- Vignette datasets (microbeAbund, SnpFile, CovFile) are simulated and not factual, so example results are illustrative only

## Evidence

- [intro] Linear regression via MatrixEQTL is mbQTL's Approach A: "A) linear regression analysis which uses core `MatrixeQTL` model."
- [intro] Inputs and covariate structure for linearTaxaSnp: "The input files are microbial abundance file (rownames taxa, colnames sample names) and SnpFile (rownames snps and colnames sample names) with optional "CovFile""
- [intro] Covariates are used for the linear regression section: "and the "covariates.txt" file is used for section A)linear regression."
- [intro] P value diagnostics and multiple-testing reporting: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
- [readme] Installation route for running the skill: [evidence span withheld — anchor status: external_doc]
- [intro] Vignette datasets are simulated: "The datasets used for this vignette are all simulated and are not factual."
