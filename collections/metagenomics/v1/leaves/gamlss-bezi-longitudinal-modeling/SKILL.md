---
name: gamlss-bezi-longitudinal-modeling
description: Use when testing differential relative abundances of taxa between groups
  (e.g. diarrhea vs. no diarrhea, breastfeeding status, gender) in microbiome relative
  abundance tables with covariate adjustment and repeated samples per subject.
when_to_use_negative:
- Input is not relative abundance data (e.g. counts, compositional transforms already
  applied, or alpha diversity indexes — use alpha.compare or other methods instead)
- The goal is pooling effect estimates across multiple studies — use meta.taxa/metatab.show
  on taxa.compare outputs instead of running taxa.compare per stratum
- The comparison variable distribution does not resemble a zero-inflated beta (e.g.
  the article notes arcsin square root transformation approaches like MaAsLin address
  continuous-transformed data, not zero inflation)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare() (GAMLSS-BEZI fitting), taxcomtab.show() (result formatting),
    and taxa.filter()/taxa.meansdn()/taxa.mean.plot() (filtering and plotting) used
    for this skill
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: gamlss
  role: Underlying GAMLSS fitting engine with the zero-inflated beta (BEZI) family
    used via propmed.rel='gamlss' in taxa.compare
- name: R
  role: Programming environment (R 3.4.2 or higher required)
- name: knitr
  role: kable() used to render the taxcomtab.show() comparison tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:96e1e6286538132f05651a9bbd1e3cf51884ce1b024a9eee11a12c07d6719f50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - gamlss
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gamlss-bezi-longitudinal-modeling

## Summary

Differential abundance testing of microbiome relative abundances using GAMLSS with a zero-inflated beta (BEZI) family in metamicrobiomeR, optionally modeling repeated samples per infant via longitudinal='yes', with FDR correction. This directly models the zero-inflated beta distribution of relative abundance data instead of applying transformations.

## When to use

Use when testing differential relative abundances of taxa between groups (e.g. diarrhea vs. no diarrhea, breastfeeding status, gender) in microbiome relative abundance tables with covariate adjustment and repeated samples per subject.

## When NOT to use

- Input is not relative abundance data (e.g. counts, compositional transforms already applied, or alpha diversity indexes — use alpha.compare or other methods instead)
- The goal is pooling effect estimates across multiple studies — use meta.taxa/metatab.show on taxa.compare outputs instead of running taxa.compare per stratum
- The comparison variable distribution does not resemble a zero-inflated beta (e.g. the article notes arcsin square root transformation approaches like MaAsLin address continuous-transformed data, not zero inflation)

## Inputs

- Taxa relative abundance table (e.g. taxtab.rm7 or taxtab6.rm from metamicrobiomeR data(), or QIIME L2-L6 output like Subramanian_et_al_mapping_file_L6.txt)
- Sample/mapping metadata with group variable (comvar), covariates (adjustvar e.g. age.sample, bf), and subject/sample IDs (personid, sampleid) for longitudinal modeling

## Outputs

- taxa.compare result object with per-taxon estimates (log odds ratio), standard errors, and FDR-adjusted p-values per comparison variable
- Formatted display table from taxcomtab.show() (via knitr::kable) at a chosen taxonomic level, filtered by p.cutoff

## How to apply

Load a taxa relative abundance table (e.g. data(taxtab.rm7) from metamicrobiomeR) and optionally filter with taxa.filter (percent.filter=0.05, relabund.filter=0.00005). Run taxa.compare() with propmed.rel='gamlss' to select the zero-inflated beta family, setting comvar to the group variable of interest, adjustvar to covariates such as age.sample, and longitudinal='yes' to account for repeated samples per infant; use p.adjust.method='fdr'. Display results at a chosen taxonomic level with taxcomtab.show() (tax.lev='l2' for phylum through 'l6' for genus), typically with readjust.p=TRUE and a p.cutoff threshold. Rationale: GAMLSS-BEZI directly addresses the zero-inflated beta distribution of relative abundances, and its estimates are log(odds ratio) of relative abundances between comparison groups, making them analogous across microbiome studies.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare() (GAMLSS-BEZI fitting), taxcomtab.show() (result formatting), and taxa.filter()/taxa.meansdn()/taxa.mean.plot() (filtering and plotting) used for this skill) — https://github.com/nhanhocu/metamicrobiomeR
- **gamlss** (Underlying GAMLSS fitting engine with the zero-inflated beta (BEZI) family used via propmed.rel='gamlss' in taxa.compare)
- **R** (Programming environment (R 3.4.2 or higher required))
- **knitr** (kable() used to render the taxcomtab.show() comparison tables)

## Examples

```
taxacom6.zi.rmg <- taxa.compare(taxtab=taxtab6.rm[[5]], propmed.rel="gamlss", comvar="bf", adjustvar="age.sample", longitudinal="yes", p.adjust.method="fdr")
```

## Evaluation signals

- Output table contains per-taxon estimates interpretable as log(odds ratio) of relative abundances between comparison groups
- FDR-adjusted p-values (p.adjust.method='fdr') flag expected taxa: in the Bangladesh diarrhea/EBF analysis, Firmicutes increased (estimate 0.49, FDR p=0.0109) and Actinobacteria decreased (estimate -0.73, FDR p=0.0011) in the EBF<=2 months stratum, with no FDR-significant phyla in the EBF>2 months stratum
- taxcomtab.show() is called with a valid tax.lev ('l2' phylum, 'l4' order, 'l5' family, 'l6' genus) and readjust.p=TRUE with p.adjust.method='fdr'
- Longitudinal flag: taxa.compare invoked with longitudinal='yes' whenever the same infant contributes multiple stool samples
- Replicated example: taxa.compare(taxtab=taxtab6.rm[[5]], propmed.rel='gamlss', comvar='bf', adjustvar='age.sample', longitudinal='yes', p.adjust.method='fdr') reproduces the README breastfeeding-status comparison

## Limitations

- Requires R 3.4.2 or higher; the CRAN version is recommended over the GitHub version as there are minor changes/updates
- GAMLSS-BEZI estimates are log(odds ratio) of relative abundances, so they are effect sizes on a specific scale and must be interpreted accordingly (backtransform=FALSE in downstream meta-analysis)
- Widely used alternatives (t-tests, Wilcoxon, linear/linear mixed models, arcsin square root transformation as in MaAsLin) do not address the zero-inflated beta distribution, so results will differ
- Additional file 1 (detailed tutorial) is referenced but not inline-accessible in the article text

## Evidence

- [abstract] GAMLSS-BEZI estimates are log(odds ratio) of relative abundances between comparison groups, making them analogous across microbiome studies: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [methods] taxa.compare GAMLSS invocation: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] taxcomtab.show display call: "kable(taxcomtab.show(taxcomtab=taxacom6.zi.rmg,tax.select=p.bf.l2$taxuse.rm, showvar="bfNon_exclusiveBF", tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr",p.cutoff = 1))"
- [intro] GAMLSS-BEZI advantage over transformation-based methods: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [discussion] GAMLSS-BEZI directly addresses relative abundance distribution: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [readme] taxa.filter thresholds applied before comparison: "taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005)"
