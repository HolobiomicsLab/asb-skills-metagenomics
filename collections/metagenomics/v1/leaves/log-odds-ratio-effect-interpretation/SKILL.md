---
name: log-odds-ratio-effect-interpretation
description: Use when reading or reporting taxa.compare()/taxcomtab.show() output
  from metamicrobiomeR GAMLSS-BEZI fits, to translate beta estimates and FDR p-values
  into directional, group-comparable effect statements about relative abundances.
when_to_use_negative:
- Estimates come from non-GAMLSS methods (e.g. MaAsLin arcsin square-root transformed
  linear models) whose coefficients are not log(odds ratio) of relative abundances
- Only a single taxon's raw abundance difference is needed without group-level odds-ratio
  framing or cross-study comparability
- Relative abundance data have not been filtered/normalized for compositional effects,
  so BEZI assumptions on the zero-inflated beta distribution do not hold
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3307
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare() with propmed.rel="gamlss" (GAMLSS-BEZI) and taxcomtab.show()
    whose estimates are log(odds ratio) of relative abundances between comparison
    groups
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Runtime environment (R 3.4.2 or higher) for loading taxtab.rm7 and running
    the GAMLSS-BEZI comparison workflow
- name: gamlss
  role: Zero-inflated beta (BEZI) family model engine underlying the differential
    abundance estimates
- name: knitr
  role: kable() display of the taxcomtab.show() effect tables used for interpretation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:0496c0efa034b3110ddaccce3fb5c46a17976c08c896ea583aa2d1448b233dd4
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Log-odds ratio effect interpretation

## Summary

Interprets GAMLSS-BEZI differential relative abundance coefficients as log(odds ratio) of relative abundances between comparison groups (e.g. diarrhea vs no diarrhea, EBF <= 2 months vs > 2 months), which makes estimates analogous across microbiome studies and suitable for meta-analysis.

## When to use

Use when reading or reporting taxa.compare()/taxcomtab.show() output from metamicrobiomeR GAMLSS-BEZI fits, to translate beta estimates and FDR p-values into directional, group-comparable effect statements about relative abundances.

## When NOT to use

- Estimates come from non-GAMLSS methods (e.g. MaAsLin arcsin square-root transformed linear models) whose coefficients are not log(odds ratio) of relative abundances
- Only a single taxon's raw abundance difference is needed without group-level odds-ratio framing or cross-study comparability
- Relative abundance data have not been filtered/normalized for compositional effects, so BEZI assumptions on the zero-inflated beta distribution do not hold

## Inputs

- taxa.compare() output object (taxcomtab) from metamicrobiomeR fitted with propmed.rel="gamlss"
- Rarefied taxa relative abundance table (e.g. data(taxtab.rm7) phylum-level component, taxtab.rm[[5]])
- Mapping/metadata variable defining the comparison group (e.g. diarrhea status, bf, gender) and adjustment covariates (e.g. age.sample)

## Outputs

- Formatted effect table from taxcomtab.show() with per-taxon log(odds ratio) estimates, standard errors, raw and FDR-adjusted p-values (e.g. phylum-level Table 5)
- Directional interpretation statements (e.g. significant Firmicutes increase and Actinobacteria decrease with diarrhea in EBF <= 2 months infants)

## How to apply

Run taxa.compare() with propmed.rel="gamlss" (zero-inflated beta BEZI family), the comparison variable of interest (e.g. diarrhea status or breastfeeding status), adjustment covariates (e.g. age.sample), longitudinal="yes" for repeated samples per infant, and p.adjust.method="fdr". Format results with taxcomtab.show() at the desired taxonomic level (tax.lev="l2" for phylum) with readjust.p=TRUE and showvar set to the comparison-group level (e.g. "bfNon_exclusiveBF"). Each GAMLSS-BEZI estimate is a log(odds ratio) of relative abundance between the two groups: a positive estimate (e.g. Firmicutes, 0.49, FDR p=0.0109) means increased relative abundance in the comparison group, a negative estimate (e.g. Actinobacteria, -0.73, FDR p=0.0011) means decreased relative abundance. Because estimates are log(odds ratio) of relative abundances, they are analogous between microbiome studies and can be pooled directly by random-effects meta-analysis (meta.taxa with summary.measure="RR", backtransform=FALSE). Verify interpretation by checking the displayed coefficient corresponds to the named comparison-group level and that FDR-adjusted p-values pass your cutoff (e.g. Table 5 pattern: significant phyla only in the EBF <= 2 months stratum).

## Related tools

- **metamicrobiomeR** (Provides taxa.compare() with propmed.rel="gamlss" (GAMLSS-BEZI) and taxcomtab.show() whose estimates are log(odds ratio) of relative abundances between comparison groups) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Runtime environment (R 3.4.2 or higher) for loading taxtab.rm7 and running the GAMLSS-BEZI comparison workflow)
- **gamlss** (Zero-inflated beta (BEZI) family model engine underlying the differential abundance estimates)
- **knitr** (kable() display of the taxcomtab.show() effect tables used for interpretation)

## Examples

```
taxacom<-taxa.compare(taxtab=taxtab.rm7[[5]],propmed.rel="gamlss",comvar="diarrhea",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr"); kable(taxcomtab.show(taxcomtab=taxacom,tax.select="none",showvar="diarrheaYes",tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr"))
```

## Evaluation signals

- Each reported estimate must be labeled/interpreted as log(odds ratio) of relative abundance between the named comparison groups
- Direction check: positive estimate = increased relative abundance in the comparison-group level passed to showvar (e.g. Firmicutes 0.49 with diarrhea in EBF <= 2 months); negative estimate = decrease (e.g. Actinobacteria -0.73)
- Significance check: FDR-adjusted p-values below cutoff only for reported taxa (FDR p=0.0109 Firmicutes, p=0.0011 Actinobacteria in EBF <= 2 months; none in EBF > 2 months)
- showvar level in taxcomtab.show() must match the comparison-group level whose coefficient is displayed (e.g. "bfNon_exclusiveBF")
- readjust.p=TRUE with p.adjust.method="fdr" was applied so displayed p-values are FDR-corrected across taxa within the level

## Limitations

- Estimates are on the log(odds ratio) scale and not backtransformed by default; meta-analysis pooling uses backtransform=FALSE
- Interpretation depends on the BEZI zero-inflated beta distributional assumption; linear-model methods on arcsin square-root transformed data do not yield comparable log(odds ratio) estimates
- Significance can be stratum-dependent: in the Bangladesh cohort the diarrhea effects were significant only in the EBF <= 2 months stratum, not in EBF > 2 months
- Adjusted estimates should be read together with covariates (e.g. age at sample collection) and longitudinal structure; ignoring repeated samples changes estimates

## Evidence

- [abstract] the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [methods] taxa.compare with gamlss and FDR adjustment: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] taxcomtab.show at phylum level with FDR readjustment: "taxcomtab.show(taxcomtab=taxacom6.zi.rmg,tax.select=p.bf.l2$taxuse.rm, showvar="bfNon_exclusiveBF", tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr",p.cutoff = 1)"
- [results] In infants with EBF duration <= 2 months, GAMLSS-BEZI detects a significant increase in Firmicutes (estimate 0.49, FDR p=0.0109) and decrease in Actinobacteria (estimate -0.73, FDR p=0.0011) with diarrhea: [evidence span withheld — anchor status: not_found]
- [discussion] GAMLSS-BEZI directly addresses the relative abundance distribution: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [readme] README recommends CRAN version and points to tutorial: "The most updated version of the package "metamicrobiomeR" is in R cran at <https://cran.r-project.org/web/packages/metamicrobiomeR/index.html>"
