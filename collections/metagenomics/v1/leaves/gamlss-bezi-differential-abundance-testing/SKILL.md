---
name: gamlss-bezi-differential-abundance-testing
description: Use when testing between-group differences in microbiome taxon relative
  abundances (phylum–genus levels) from compositional data with many zeros, especially
  with covariate adjustment and repeated-measures/longitudinal designs.
when_to_use_negative:
- Input is raw counts or OTU/ASV feature counts rather than relative abundances —
  GAMLSS-BEZI models compositional relative abundance data and expects transformed/normalized
  proportions
- The analysis goal is pooling effect estimates across studies — use meta.taxa random-effects
  meta-analysis instead of a single-cohort taxa.compare test
- The question concerns alpha diversity indexes or microbiome age prediction — use
  alpha.compare or the Random Forest microbiomeage workflow, not taxa.compare
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare implementing GAMLSS with zero-inflated beta (BEZI) family
    for differential relative abundance testing, plus taxa.filter/taxa.meansdn/taxcomtab.show
    for filtering, summarizing, and display
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Runtime environment (R 3.4.2 or higher required) for the metamicrobiomeR package
- name: GAMLSS-BEZI
  role: Zero-inflated beta distribution family used to model taxon relative abundances
    directly in the regression
- name: MaAsLin
  role: 'Baseline comparator: linear model with arcsin square root transformation
    (LMAS) used to benchmark type I error and power'
- name: devtools
  role: Used to install the package from GitHub via install_github('nhanhocu/metamicrobiomeR')
- name: knitr
  role: Renders comparison result tables via knitr::kable
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  - task_006
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:d1af16715abef1e5f6dbb3c2da58040f99c34c556749656989bf6f168d0d8650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - MaAsLin
  - R
  - devtools
  - gamlss
  - gdata
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# gamlss-bezi-differential-abundance-testing

## Summary

Differential relative abundance testing of microbial taxa using zero-inflated beta GAMLSS (BEZI family) via metamicrobiomeR's taxa.compare, which directly models the zero-inflated beta distribution of microbiome relative abundance data instead of transforming it (as in MaAsLin's arcsin square-root linear models).

## When to use

Use when testing between-group differences in microbiome taxon relative abundances (phylum–genus levels) from compositional data with many zeros, especially with covariate adjustment and repeated-measures/longitudinal designs.

## When NOT to use

- Input is raw counts or OTU/ASV feature counts rather than relative abundances — GAMLSS-BEZI models compositional relative abundance data and expects transformed/normalized proportions
- The analysis goal is pooling effect estimates across studies — use meta.taxa random-effects meta-analysis instead of a single-cohort taxa.compare test
- The question concerns alpha diversity indexes or microbiome age prediction — use alpha.compare or the Random Forest microbiomeage workflow, not taxa.compare

## Inputs

- Taxon relative abundance table by taxonomic level (e.g. taxtab.rm7 list; l2 phylum, l5 family, l6 genus)
- Sample metadata/mapping file with comparison, adjustment, and person-ID variables (e.g. sam.rm with bf, age.sample, gender, personid)
- Optional processed 16S data (e.g. Subramanian et al 2014 Bangladesh cohort from gordonlab.wustl.edu)

## Outputs

- Per-taxon comparison table with estimates (log odds ratios), standard errors, 95% CIs, and FDR-adjusted p-values (e.g. bf_phylum_comparison.csv)
- Rendered kable tables of significant taxa per taxonomic level (taxcomtab.show output)

## How to apply

Run taxa.compare from metamicrobiomeR with propmed.rel='gamlss' so relative abundances are modeled with the GAMLSS zero-inflated beta (BEZI) family, specifying comvar as the comparison variable (e.g. 'bf' or 'gender'), adjustvar for covariates (e.g. 'age.sample', c('bf','age.sample')), and longitudinal='yes' with a person-level ID for repeated measures. Filter taxa first with taxa.filter (percent.filter=0.05, relabund.filter=0.00005) and restrict/summarize inputs with taxa.meansdn when exploring by group. Adjust p-values with p.adjust.method='fdr' and display results via taxcomtab.show (e.g. showvar='bfNon_exclusiveBF', tax.lev='l2', readjust.p=TRUE, p.cutoff=1 to view all). Estimates are log(odds ratio) of relative abundances, which makes them analogous across microbiome studies and poolable by random-effects meta-analysis. Validate by comparing type I error control and power against the arcsin square-root linear model (LMAS/MaAsLin) baseline: simulations with BEZI-generated data (mu1=0.5, alpha=0.5, phi=5, n=100/group, 1000 replicates) showed GAMLSS-BEZI was well calibrated and more powerful at every effect size (mu2=0.4, 0.3, 0.2).

## Related tools

- **metamicrobiomeR** (Provides taxa.compare implementing GAMLSS with zero-inflated beta (BEZI) family for differential relative abundance testing, plus taxa.filter/taxa.meansdn/taxcomtab.show for filtering, summarizing, and display) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Runtime environment (R 3.4.2 or higher required) for the metamicrobiomeR package)
- **GAMLSS-BEZI** (Zero-inflated beta distribution family used to model taxon relative abundances directly in the regression)
- **MaAsLin** (Baseline comparator: linear model with arcsin square root transformation (LMAS) used to benchmark type I error and power)
- **devtools** (Used to install the package from GitHub via install_github('nhanhocu/metamicrobiomeR'))
- **knitr** (Renders comparison result tables via knitr::kable)

## Examples

```
taxacom6.zi.rmg <- taxa.compare(taxtab=taxtab6.rm[[5]], propmed.rel="gamlss", comvar="bf", adjustvar="age.sample", longitudinal="yes", p.adjust.method="fdr")
```

## Evaluation signals

- Type I error rates from BEZI-simulated null data (mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5; n=10/100/500; 1000 replicates) fall within ~2 binomial standard errors of nominal alpha (0.01/0.05/0.1)
- Power under simulated alternatives (mu2=0.4/0.3/0.2, n=100/group) monotonically increases with effect size and exceeds the LMAS baseline at each effect size
- Output table rows have numeric estimate, 95% CI, and FDR-adjusted p-values with p in [0,1] and readjustment applied via taxcomtab.show(readjust.p=TRUE, p.adjust.method='fdr')
- Estimates are interpretable as log(odds ratio) of relative abundances between comparison groups
- On real Bangladesh cohort data, GAMLSS-BEZI detects more significant phyla (e.g. all four phyla in non-EBF vs EBF; three in solid-food timing) than LMAS (three and one respectively)

## Limitations

- Widely used methods this skill replaces (t-tests, Wilcoxon, linear/linear mixed models) exist because many analysts default to them; arcsin square-root transformation (MaAsLin) does not address zero inflation
- The GitHub-hosted version has minor changes compared with the recommended CRAN version; users are recommended to use the CRAN version
- GAMLSS-BEZI fits can be computationally demanding on large taxa tables and longitudinal designs (e.g. 996-sample Bangladesh longitudinal subset with 1000 random splits)

## Evidence

- [abstract] GAMLSS-BEZI for differential abundance testing: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [abstract] log odds ratio estimates: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [intro] transformation does not address zeros: "However, this adjustment does not address the inflation of zero values in microbiome relative abundance data."
- [methods] taxa.compare invocation: "taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] taxa.filter thresholds: "taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005)"
- [readme] README example workflow: "The *metamicrobiomeR* package implements Generalized Additive Model for Location, Scale and Shape (GAMLSS) with zero inflated beta (BEZI) family for analysis of microbiome relative abundance data"
