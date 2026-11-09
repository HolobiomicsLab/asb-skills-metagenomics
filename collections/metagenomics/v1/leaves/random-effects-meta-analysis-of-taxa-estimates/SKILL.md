---
name: random-effects-meta-analysis-of-taxa-estimates
description: Use when you have comparable per-study differential taxa comparison results
  (estimates + standard errors from GAMLSS-BEZI taxa.compare) from ≥2 microbiome studies
  and need pooled overall effects and heterogeneity across studies.
when_to_use_negative:
- You only have a single study — no pooling is needed; use taxa.compare directly with
  FDR adjustment instead
- The per-study estimates are not on a comparable scale (not GAMLSS-BEZI log odds
  ratios of relative abundances), so pooling would be invalid
- The goal is alpha diversity comparison or microbiome age prediction, which use alpha.compare
  or microbiomeage functions rather than meta.taxa
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides meta.taxa for random-effects pooling, metatab.show for filtered result
    tables, and meta.niceplot for heatmap-forest visualization
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Programming language in which the meta-analysis workflow is executed (R 3.4.2
    or higher)
- name: devtools
  role: Used to install the metamicrobiomeR package from GitHub via install_github('nhanhocu/metamicrobiomeR')
- name: knitr
  role: Used with kable() to render the pooled meta-analysis tables
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:51be2fb3765419a5854de0c1a433ccecf07753ae0911021f9df654a6d147eb1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - devtools
  - knitr
  - metamicrobiomeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# random-effects-meta-analysis-of-taxa-estimates

## Summary

Pool per-study GAMLSS-BEZI differential taxa estimates (log odds ratios of relative abundances) and their standard errors across microbiome studies using random-effects meta-analysis in the metamicrobiomeR package, producing pooled effect estimates, FDR-adjusted p-values, and heatmap-forest displays.

## When to use

Use when you have comparable per-study differential taxa comparison results (estimates + standard errors from GAMLSS-BEZI taxa.compare) from ≥2 microbiome studies and need pooled overall effects and heterogeneity across studies.

## When NOT to use

- You only have a single study — no pooling is needed; use taxa.compare directly with FDR adjustment instead
- The per-study estimates are not on a comparable scale (not GAMLSS-BEZI log odds ratios of relative abundances), so pooling would be invalid
- The goal is alpha diversity comparison or microbiome age prediction, which use alpha.compare or microbiomeage functions rather than meta.taxa

## Inputs

- Per-study taxa comparison result tables from taxa.compare with GAMLSS-BEZI estimates and standard errors (e.g. taxacom.rm.sex.adjustbfage, taxacom.ha.sex.adjustbfage, taxacom6.zi.usbmk.sex.adjustbfage, taxacom6.unc.sex.adjustedbfage)
- Combined pooled comparison table (tabsex4) with study and population labels
- metamicrobiomeR package installed in R (>= 3.4.2)

## Outputs

- Random-effects meta-analysis result table (metab.sex$random) with pooled estimates, standard errors, pooled p-values, and FDR-adjusted p-values per taxon
- Genus-level pooled results CSV (meta_taxa_gender_results.csv)
- Heatmap-forest figures at phylum (level='main') and family/genus (level='sub') levels from meta.niceplot

## How to apply

First run taxa.compare (propmed.rel='gamlss') per study to obtain group-comparison estimates, which are log(odds ratio) of relative abundances and thus analogous across studies. Tag each study's result table with study and population labels (e.g. 'Subramanian et al 2014 (Bangladesh)') and combine the tables with plyr::rbind.fill into a single pooled comparison table. Pool estimates with meta.taxa using summary.measure='RR', pool.var='id', studylab='study', backtransform=FALSE, percent.meta=0.5, and p.adjust.method='fdr'. Extract result tables at phylum (l2), family (l5), and genus (l6) levels with metatab.show (showvar='genderMale', p.cutoff.type='p', p.cutoff=0.05, display='table') and render heatmap-forest plots with meta.niceplot (sumtype='taxa', level='main' for phylum, level='sub' for family/genus, phyla.col='rainbow', p='p', p.adjust='p.adjust'). The random-effects model directly pools the adjusted estimates and their standard errors to evaluate overall effects and between-study heterogeneity.

## Related tools

- **metamicrobiomeR** (Provides meta.taxa for random-effects pooling, metatab.show for filtered result tables, and meta.niceplot for heatmap-forest visualization) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Programming language in which the meta-analysis workflow is executed (R 3.4.2 or higher))
- **devtools** (Used to install the metamicrobiomeR package from GitHub via install_github('nhanhocu/metamicrobiomeR'))
- **knitr** (Used with kable() to render the pooled meta-analysis tables)

## Examples

```
metab.sex<-meta.taxa(taxcomdat=tabsex4,summary.measure="RR",pool.var="id",studylab="study",backtransform=FALSE,percent.meta=0.5,p.adjust.method="fdr")
```

## Evaluation signals

- The pooled phylum-level table (tax.lev='l2', p.cutoff=0.05, FDR-adjusted) shows no significant gender differences between male and female infants
- The genus-level pooled table flags Coprococcus as significantly higher in males (FDR adjusted pooled p-value < 0.0001), matching the reported meta-analysis result
- No KEGG pathway remains significant after FDR adjustment in the pooled functional-pathway analysis
- Every input study table carries correct study and population labels (Bangladesh, Haiti, USA(CA_FL), USA(NC)) and the combined table rows equal the sum of the four study tables
- Random-effects output includes pooled estimates and standard errors plus between-study heterogeneity measures for each taxon id

## Limitations

- Meta-analysis requires estimates on a common scale; GAMLSS-BEZI log(odds ratio) estimates are analogous across studies, but estimates from other differential abundance methods may not be poolable
- Existing microbiome meta-analysis approaches that transform relative abundances (e.g. arcsin square root as in MaAsLin) do not address the zero-inflated distribution, limiting comparability
- The GitHub version has minor differences from the recommended CRAN version of metamicrobiomeR; users are recommended to use the CRAN version

## Evidence

- [abstract] Random effects meta-analysis models can be directly applied to pool comparable estimates and their standard errors to evaluate overall effects and heterogeneity: [evidence span withheld — anchor status: not_found]
- [abstract] GAMLSS-BEZI estimates are log odds ratios: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [methods] meta.taxa invocation in methods: "metab.sex<-meta.taxa(taxcomdat=tabsex4,summary.measure="RR",pool.var="id",studylab="study",backtransform=FALSE,percent.meta=0.5,p.adjust.method="fdr")"
- [readme] Study labelling and combining: "taxacom6.zi.rm.sex.adjustbfage$study<-"Subramanian et al 2014 (Bangladesh)""
- [methods] metatab.show display parameters: "kable(metatab.show(metatab=metab.sex$random,com.pooled.tab=tabsex4,tax.lev="l2",showvar="genderMale",p.cutoff.type="p", p.cutoff=0.05,display="table"))"
- [readme] meta.niceplot heatmap-forest display: "meta.niceplot(metadat=metadat,sumtype="taxa",level="main",p="p",p.adjust="p.adjust",phyla.col="rainbow",forest.col="by.estimate""
