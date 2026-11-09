---
name: binary-presence-absence-taxon-modeling
description: Use when microbial abundance is treated as a binary presence/absence
  trait and you need to estimate the relationship of one taxon to one SNP (genotype)
  across a large sample cohort.
when_to_use_negative:
- When you need continuous taxon abundance modeled via linear regression — use mbQTL's
  MatrixEQTL-based approach (Approach A) instead of the categorical logistic regression
  of Approach C.
- When your goal is rho estimation for taxa–SNP correlation after taxa–taxa R values
  (Approach B), which addresses linkage disequilibrium SNPs rather than presence/absence
  associations.
- When the input is not an abundance table with taxa as rownames and samples as colnames
  (e.g. an unnormalized metagenomeSeq or phyloseq object) — first normalize/convert
  via metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq().
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3174
tools:
- name: mbQTL
  role: R/Bioconductor package providing logRegSnpsTaxa (and its selectmicrobe variant)
    for logistic regression of presence/absence taxa against SNPs
  repo: https://github.com/Mercedeh66/mbQTL
- name: R
  role: Runtime environment in which mbQTL functions are called and simulated datasets
    are loaded via data()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad565
    title: 'mbQTL: an R/Bioconductor package for microbial quantitative trait loci
      (QTL) estimation'
  source_content_hash: sha256:07f087426bb0fee60967819704e844f697d2a4eca50e8fc6ecb3578c002fe238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mercedeh66/mbQTL
  tools:
  - R
  - mbQTL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# binary-presence-absence-taxon-modeling

## Summary

This skill applies mbQTL's Approach C: logistic regression between one taxon and one SNP using presence/absence (categorical) taxon status, implemented via the logRegSnpsTaxa function in the mbQTL R/Bioconductor package, to estimate microbe–SNP relationships across a sample cohort.

## When to use

Use when microbial abundance is best treated as a binary presence/absence trait and you need to estimate the relationship of one taxon to one SNP (genotype) across a large sample cohort.

## When NOT to use

- When you need continuous taxon abundance modeled via linear regression — use mbQTL's MatrixEQTL-based approach (Approach A) instead of the categorical logistic regression of Approach C.
- When your goal is rho estimation for taxa–SNP correlation after taxa–taxa R values (Approach B), which addresses linkage disequilibrium SNPs rather than presence/absence associations.
- When the input is not an abundance table with taxa as rownames and samples as colnames (e.g. an unnormalized metagenomeSeq or phyloseq object) — first normalize/convert via metagenomeSeqToMbqtl or phyloseq_to_metagenomeSeq().

## Inputs

- microbeAbund: microbial abundance table (rownames taxa, colnames sample names), e.g. the mbQTL simulated data("microbeAbund")
- SnpFile: SNP genotype table (rownames snps, colnames sample names), e.g. data("SnpFile")
- Optional single-taxon selection, e.g. selectmicrobe = "Haemophilus"

## Outputs

- Table of logistic regression results for taxon–SNP pairs (coefficients and statistical measures per pair), exportable as e.g. logistic_regression_taxa_snp_results.csv
- One-taxon-vs-one-SNP result when the selectmicrobe variant is used

## How to apply

Load the mbQTL input objects in R: microbeAbund (microbial abundance table, rownames taxa, colnames samples) and SnpFile (SNP genotype table, rownames SNPs, colnames samples). Call logRegSnpsTaxa(microbeAbund, SnpFile) to fit, for each taxon–SNP pair, a logistic regression model in which the taxon is recoded as a presence/absence (categorical) response across the sample cohort. To restrict the analysis to a single taxon such as Haemophilus, use the selectmicrobe variant of logRegSnpsTaxa. Collect the per-pair logistic regression coefficients and statistical measures, writing significant taxon–SNP results to a CSV file. Rationale: logistic regression is the appropriate formulation when taxa abundance is summarized categorically rather than continuously, complementing mbQTL's MatrixEQTL-based linear regression and rho-correlation approaches.

## Related tools

- **mbQTL** (R/Bioconductor package providing logRegSnpsTaxa (and its selectmicrobe variant) for logistic regression of presence/absence taxa against SNPs) — https://github.com/Mercedeh66/mbQTL
- **R** (Runtime environment in which mbQTL functions are called and simulated datasets are loaded via data())

## Examples

```
data("microbeAbund"); data("SnpFile"); res <- logRegSnpsTaxa(microbeAbund, SnpFile); res_Haemophilus <- logRegSnpsTaxa(microbeAbund, SnpFile, selectmicrobe = "Haemophilus")
```

## Evaluation signals

- Output table contains fitted logistic regression statistics for taxon–SNP pairs, not raw abundances
- The Haemophilus-restricted result (selectmicrobe variant) is present in the output alongside genome-wide results
- The analysis can be regenerated from the packaged simulated data (data("microbeAbund"); data("SnpFile")) without error
- Input tables conform to mbQTL expectations: taxa/SNPs as rownames and sample names as colnames
- Significant taxon–SNP pairs are written to logistic_regression_taxa_snp_results.csv with per-pair coefficients and statistical measures

## Limitations

- Approach C uses presence/absence (categorical) taxa, so continuous abundance variation within a taxon is not modeled; use the linear regression approach for continuous abundance.
- The paper's vignette datasets are simulated: "The datasets used for this vignette are all simulated and are not factual."
- Covariates (CovFile) are described for the linear regression section, not shown as part of the logistic regression call in the provided workflow.

## Evidence

- [other] Approach C logistic regression between taxa and SNPs is performed with the logRegSnpsTaxa function: [evidence span withheld — anchor status: not_found]
- [intro] Logistic regression estimates the relationship of one snp and one taxa with categorical presence/absence taxa: "Utilizing Logistic regression analysis, we estimate the relationship of one snp and one taxa across the sample cohort"
- [intro] Vignette datasets are simulated: "The datasets used for this vignette are all simulated and are not factual."
- [intro] Input file formats: microbial abundance with taxa rownames and SnpFile with snp rownames: "The input files are microbial abundance file (rownames taxa, colnames sample names)"
- [intro] mbQTL identifies significant taxa-SNP relationships across large sample sets: "`mbQTL` is a statistical package for identifying significant taxa (microbial), snp (genotype) relationships across large sample sets."
