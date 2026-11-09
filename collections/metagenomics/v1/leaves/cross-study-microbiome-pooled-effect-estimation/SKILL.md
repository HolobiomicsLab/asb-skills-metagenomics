---
name: cross-study-microbiome-pooled-effect-estimation
description: Use when you have comparable per-study differential abundance result
  tables (estimates + standard errors from GAMLSS-BEZI taxa.compare on the same taxa
  levels) and need pooled effects, heterogeneity, and FDR-adjusted p-values across
  studies.
when_to_use_negative:
- Input is raw microbiome relative abundance data rather than per-study comparison
  result tables — run per-study taxa.compare with GAMLSS-BEZI first
- Studies produced estimates with methods that are not analogous across studies (e.g.
  arcsin square-root transformed linear models as in MaAsLin), since log(odds ratio)
  comparability is what makes pooling valid
- Interest is in a single-study differential abundance test rather than cross-study
  pooled effects and heterogeneity
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3303
tools:
- name: metamicrobiomeR
  role: Provides meta.taxa (random-effects pooling of estimates/SEs with summary.measure='RR'
    and percent.meta=0.5), metatab.show (pooled result tables), and meta.niceplot
    (heatmap-forest figures)
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Runtime language; requires R 3.4.2 or higher
- name: devtools
  role: Used to install the package from GitHub via install_github('nhanhocu/metamicrobiomeR')
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:28f0fece29cae3d7d4607043512f0a912ecabe43ed2fed1f360bde9ee841b432
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - devtools
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cross-study-microbiome-pooled-effect-estimation

## Summary

Pools per-study GAMLSS-BEZI differential abundance estimates (log odds ratios of taxa relative abundances) across microbiome studies using random-effects meta-analysis in metamicrobiomeR, as demonstrated for the four-study infant gender comparison (Subramanian 2014 Bangladesh, Bender 2016 Haiti, Pannaraj 2017 USA(CA_FL), Thompson 2015 USA(NC)).

## When to use

Use when you have comparable per-study differential abundance result tables (estimates + standard errors from GAMLSS-BEZI taxa.compare on the same taxa levels) and need pooled effects, heterogeneity, and FDR-adjusted p-values across studies.

## When NOT to use

- Input is raw microbiome relative abundance data rather than per-study comparison result tables — run per-study taxa.compare with GAMLSS-BEZI first
- Studies produced estimates with methods that are not analogous across studies (e.g. arcsin square-root transformed linear models as in MaAsLin), since log(odds ratio) comparability is what makes pooling valid
- Interest is in a single-study differential abundance test rather than cross-study pooled effects and heterogeneity

## Inputs

- Per-study taxa comparison result tables from GAMLSS-BEZI taxa.compare, e.g. taxacom.rm.sex.adjustbfage, taxacom.ha.sex.adjustbfage, taxacom6.zi.usbmk.sex.adjustbfage, taxacom6.unc.sex.adjustedbfage (package data objects)
- Taxa relative abundance tables filtered with taxa.filter (e.g. percent.filter=0.05, relabund.filter=0.00005) if recomputing per-study comparisons
- Study/population labels for each per-study table

## Outputs

- metab.sex$random: random-effects pooled results table with pooled estimates, standard errors, heterogeneity, and FDR-adjusted p-values
- Pooled taxa tables at phylum (l2), family (l5), and genus (l6) levels via metatab.show, e.g. exported as meta_taxa_gender_results.csv
- Heatmap-forest PNG figures from meta.niceplot showing pooled effects and significance by taxon

## How to apply

First run taxa.compare with propmed.rel='gamlss' per study (e.g. comvar='gender', adjustvar=c('bf','age.sample'), longitudinal='yes' on infants <=6 months), tag each table with 'study' and 'pop' labels, and combine with plyr::rbind.fill into a single pooled table. Pool with meta.taxa(taxcomdat=tabsex4, summary.measure='RR', pool.var='id', studylab='study', backtransform=FALSE, percent.meta=0.5, p.adjust.method='fdr'); the RR summary measure works because GAMLSS-BEZI estimates are log(odds ratio) and thus analogous across studies, and percent.meta=0.5 restricts pooling to taxa observed in at least 50% of studies. Generate pooled result tables at phylum (l2), family (l5), and genus (l6) levels via metatab.show with showvar='genderMale', p.cutoff.type='p', p.cutoff=0.05, display='table'. For figures, call metatab.show with p.cutoff=1 and display='data', then render heatmap-forest plots with meta.niceplot (sumtype='taxa', level='main' for phylum, level='sub' for family/genus, phyla.col='rainbow', p='p', p.adjust='p.adjust').

## Related tools

- **metamicrobiomeR** (Provides meta.taxa (random-effects pooling of estimates/SEs with summary.measure='RR' and percent.meta=0.5), metatab.show (pooled result tables), and meta.niceplot (heatmap-forest figures)) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Runtime language; requires R 3.4.2 or higher)
- **devtools** (Used to install the package from GitHub via install_github('nhanhocu/metamicrobiomeR'))

## Examples

```
metab.sex<-meta.taxa(taxcomdat=tabsex4,summary.measure="RR",pool.var="id",studylab="study",backtransform=FALSE,percent.meta=0.5,p.adjust.method="fdr")
```

## Evaluation signals

- Pooled phylum-level (l2) table via metatab.show with p.cutoff.type='p', p.cutoff=0.05 shows no significant gender differences after FDR adjustment, matching the reported result
- Genus-level (l6) pooled table flags Coprococcus as significantly higher in males (FDR-adjusted pooled p < 0.0001 in the source study)
- The combined table tabsex4 contains exactly four study labels (Subramanian et al 2014 (Bangladesh), Bender et al 2016 (Haiti), Pannaraj et al 2017 (USA(CA_FL)), Thompson et al 2015 (USA(NC))) and is built with rbind.fill without row loss on shared columns
- meta.taxa runs with summary.measure='RR', backtransform=FALSE, percent.meta=0.5, p.adjust.method='fdr' and returns pooled objects including metab.sex$random with heterogeneity estimates
- Pooled estimates are interpretable as log(odds ratio) of relative abundances; meta.niceplot receives data from metatab.show with p.cutoff=1, display='data' so all taxa appear in the figure

## Limitations

- Pooling is only valid because GAMLSS-BEZI estimates are log(odds ratio) of relative abundances and thus analogous between studies; estimates from other differential abundance methods are not directly poolable
- percent.meta=0.5 restricts meta-analysis to taxa present in at least 50% of studies, so rare taxa are dropped
- Random effects meta-analysis evaluates overall effects and between-study heterogeneity; zero inflation of relative abundances means standard t-tests/Wilcoxon/linear models on transformed data are inappropriate and would break comparability
- The GitHub version differs slightly from the recommended CRAN version of metamicrobiomeR

## Evidence

- [discussion] Random effects meta-analysis models can be directly applied to pool the adjusted estimates and their standard errors across studies.: "Random effects meta-analysis models can be directly applied to pool the adjusted estimates and their standard errors across studies."
- [readme] meta.taxa call in README: "metab.sex<-meta.taxa(taxcomdat=tabsex4,summary.measure="RR",pool.var="id",studylab="study",backtransform=FALSE,percent.meta=0.5,p.adjust.method="fdr")"
- [abstract] GAMLSS-BEZI estimates as log OR: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [methods] gender comparison workflow: "Comparison of bacterial taxa relative abundance in infants < 6 months between gender adjusting for breastfeeding statuses and age of infants at sample collection with GAMLSS"
