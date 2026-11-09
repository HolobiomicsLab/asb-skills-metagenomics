---
name: stratified-cohort-analysis-by-breastfeeding-duration
description: Use when testing a binary exposure (e.g., diarrhea status) against microbiome
  relative abundances within an exclusive-breastfeeding-duration stratum in a longitudinal
  infant cohort, expecting log(odds ratio) estimates with FDR-adjusted p-values.
when_to_use_negative:
- The goal is cross-study pooling of estimates rather than within-cohort stratified
  testing — use meta.taxa random effects meta-analysis instead
- The exposure of interest is continuous or multi-level (e.g., alpha diversity indexes
  over breastfeeding statuses) and the zero-inflated beta model for binary group comparison
  is not appropriate
- Data are counts or absolute abundances rather than relative abundances resembling
  a zero-inflated beta distribution
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare (GAMLSS-BEZI differential abundance testing), taxcomtab.show
    (formatted phylum tables with FDR readjustment), and the taxtab.rm7 Bangladesh
    example dataset
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Programming language in which the stratified GAMLSS-BEZI workflow is executed
    (R 3.4.2 or higher)
- name: gamlss
  role: Underlying zero-inflated beta (BEZI) regression engine invoked by taxa.compare
    when propmed.rel='gamlss'
- name: knitr
  role: Used to render taxcomtab.show output tables (e.g., via kable) for comparison
    against Table 5
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:3b6ea189f3bc0b83b2243825e7e6472e5f99c3372b04cbe43ab2d4c2e062ed92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - gamlss
  - knitr
  - metamicrobiomeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# stratified-cohort-analysis-by-breastfeeding-duration

## Summary

Stratify a longitudinal infant microbiome cohort (e.g., Bangladesh) by duration of exclusive breastfeeding (<=2 months vs >2 months) and, within each stratum, test differential phylum-level relative abundances (e.g., diarrhea vs no diarrhea) using GAMLSS zero-inflated beta models via metamicrobiomeR, with FDR correction.

## When to use

Use when testing a binary exposure (e.g., diarrhea status) against microbiome relative abundances within an exclusive-breastfeeding-duration stratum in a longitudinal infant cohort, expecting log(odds ratio) estimates with FDR-adjusted p-values.

## When NOT to use

- The goal is cross-study pooling of estimates rather than within-cohort stratified testing — use meta.taxa random effects meta-analysis instead
- The exposure of interest is continuous or multi-level (e.g., alpha diversity indexes over breastfeeding statuses) and the zero-inflated beta model for binary group comparison is not appropriate
- Data are counts or absolute abundances rather than relative abundances resembling a zero-inflated beta distribution

## Inputs

- Bangladesh phylum-level taxa relative abundance table (data(taxtab.rm7), taxtab.rm[[5]] component) from the metamicrobiomeR package
- Sample mapping file (e.g., sam.rm / merged samfile) with diarrhea status, exclusive breastfeeding duration in months (month.exbf), age.sample, and per-infant identifiers (personid) for repeated samples

## Outputs

- Per-stratum taxa.compare result objects containing GAMLSS-BEZI log(odds ratio) estimates, standard errors, and FDR-adjusted p-values per phylum
- Formatted phylum-level comparison tables (tax.lev='l2') from taxcomtab.show, suitable for knitr::kable rendering, mirroring Table 5

## How to apply

Load the Bangladesh phylum-level relative abundance table via data(taxtab.rm7) from metamicrobiomeR and subset samples to each EBF-duration stratum (<=2 months vs >2 months, using month.exbf from the mapping file). Within each stratum, run taxa.compare() with propmed.rel='gamlss' (BEZI zero-inflated beta family), comvar set to the binary exposure (e.g., diarrhea status), adjustvar='age.sample' for age at sample collection, and longitudinal='yes' to account for repeated samples per infant; use p.adjust.method='fdr'. Format and display results with taxcomtab.show() at tax.lev='l2' (phylum), showvar set to the exposure coefficient of interest, readjust.p=TRUE, and p.adjust.method='fdr'. GAMLSS-BEZI estimates are log(odds ratio) of relative abundances, so a positive estimate (e.g., Firmicutes, 0.49, FDR p=0.0109) means enrichment in the exposed group and a negative estimate (e.g., Actinobacteria, -0.73, FDR p=0.0011) means depletion. Reproduction is judged correct when the EBF<=2mo stratum recovers a significant positive Firmicutes and significant negative Actinobacteria coefficient, and the EBF>2mo stratum shows no FDR-significant phyla (Table 5 pattern).

## Related tools

- **metamicrobiomeR** (Provides taxa.compare (GAMLSS-BEZI differential abundance testing), taxcomtab.show (formatted phylum tables with FDR readjustment), and the taxtab.rm7 Bangladesh example dataset) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Programming language in which the stratified GAMLSS-BEZI workflow is executed (R 3.4.2 or higher))
- **gamlss** (Underlying zero-inflated beta (BEZI) regression engine invoked by taxa.compare when propmed.rel='gamlss')
- **knitr** (Used to render taxcomtab.show output tables (e.g., via kable) for comparison against Table 5)

## Examples

```
taxacom.ebf2<-taxa.compare(taxtab=taxtab_ebf2,propmed.rel="gamlss",comvar="diarrhea",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr"); kable(taxcomtab.show(taxcomtab=taxacom.ebf2,tax.select="none",showvar="diarrheaYes",tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr",p.cutoff=1))
```

## Evaluation signals

- EBF<=2mo stratum recovers significant positive Firmicutes estimate (0.49, FDR p=0.0109) and significant negative Actinobacteria estimate (-0.73, FDR p=0.0011) for diarrhea
- EBF>2mo stratum yields no phyla with FDR-significant differences after readjust.p=TRUE and p.adjust.method='fdr'
- Estimates are on the log(odds ratio) scale, consistent with GAMLSS-BEZI parameterization, so signs and magnitudes are interpretable as direction of relative-abundance change
- taxcomtab.show is called at tax.lev='l2' and only the exposure coefficient row (e.g., diarrhea variable level) is displayed per phylum
- All taxa pass the FDR readjustment check: reported p.adjust values equal a fresh fdr correction over the l2 table

## Limitations

- Results are specific to the Bangladesh cohort; the article's demonstrations cover other studies (Haiti, USA(CA_FL), USA(NC)) for gender comparisons, so stratum-specific diarrhea effects are not transferable across cohorts
- Longitudinal repeated samples per infant must be declared (longitudinal='yes'); omitting it misstates the model and invalidates standard errors
- Strata with few infants or samples may be underpowered, and the EBF>2mo stratum detected no significant phyla, which does not prove absence of biological effects
- GAMLSS-based taxa.compare can be slow to run and depends on data preprocessing (e.g., taxa.filter with percent.filter=0.05, relabund.filter=0.00005) done upstream

## Evidence

- [methods] Comparison of bacterial taxa relative abundance using GAMLSS: "Comparison of bacterial taxa relative abundance using GAMLSS"
- [methods] taxa.compare call with gamlss and longitudinal adjustment: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] taxcomtab.show phylum-level display with FDR readjustment: "kable(taxcomtab.show(taxcomtab=taxacom6.zi.rmg,tax.select=p.bf.l2$taxuse.rm, showvar="bfNon_exclusiveBF", tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr",p.cutoff = 1))"
- [abstract] GAMLSS-BEZI estimates are log odds ratios: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [readme] README recommends CRAN version and tutorial file for the workflow: "The most updated version of the package "metamicrobiomeR" is in R cran at <https://cran.r-project.org/web/packages/metamicrobiomeR/index.html> ."
