---
name: longitudinal-mixed-model-microbiome-testing
description: Use when relative abundance taxa tables (phylum-to-genus, e.g. l2/l5/l6
  levels) come from longitudinal stool sampling with repeated measures per subject
  and you need between-group differential abundance testing with proper type I error
  control.
when_to_use_negative:
- Data are counts (not compositional relative abundances) and a count-based model
  such as negative binomial is intended — GAMLSS-BEZI in metamicrobiomeR targets zero-inflated
  beta relative abundance data
- Samples are independent (no repeated measures per subject), where the longitudinal='yes'
  random-intercept structure and per-subject random effects are unnecessary
- The goal is cross-study pooling rather than within-study differential abundance
  testing — use the package's random-effects meta-analysis (meta.taxa) workflow instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare with propmed.rel='gamlss' (GAMLSS zero-inflated beta
    family) and longitudinal='yes' for differential relative abundance testing in
    longitudinal designs; install via devtools::install_github('nhanhocu/metamicrobiomeR')
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Statistical model implemented by metamicrobiomeR that directly addresses the
    zero-inflated beta distribution of microbiome relative abundance data; estimates
    are log(odds ratio) of relative abundances
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: MaAsLin
  role: Comparison method representing the linear model approach on arcsin square
    root transformed relative abundances (LMAS) used in parallel to benchmark type
    I error control
- name: knitr
  role: Used with kable() to render taxa comparison tables (taxcomtab.show) in the
    reproducible workflow
- name: R
  role: Programming language; requires R 3.4.2 or higher per package documentation
  repo: https://github.com/nhanhocu/metamicrobiomeR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:b70a4d4dbacd268866e87fa5e447054cc17abbadb58f5a2f8e37df537fedc461
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - MaAsLin
  - R
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# longitudinal-mixed-model-microbiome-testing

## Summary

A skill for testing differential relative abundance of bacterial taxa in repeated-measures (longitudinal) microbiome designs, using GAMLSS with a zero-inflated beta (BEZI) family via metamicrobiomeR::taxa.compare and, in parallel, linear/linear mixed models with random intercepts (LMAS/MaAsLin-style), and for validating type I error control on real data such as the Bangladesh infant cohort.

## When to use

Use when relative abundance taxa tables (phylum-to-genus, e.g. l2/l5/l6 levels) come from longitudinal stool sampling with repeated measures per subject and you need between-group differential abundance testing with proper type I error control.

## When NOT to use

- Data are counts (not compositional relative abundances) and a count-based model such as negative binomial is intended — GAMLSS-BEZI in metamicrobiomeR targets zero-inflated beta relative abundance data
- Samples are independent (no repeated measures per subject), where the longitudinal='yes' random-intercept structure and per-subject random effects are unnecessary
- The goal is cross-study pooling rather than within-study differential abundance testing — use the package's random-effects meta-analysis (meta.taxa) workflow instead

## Inputs

- Processed 16S relative abundance taxonomy table by taxonomic level (e.g. taxtab.rm7, levels l2 phylum through l6 genus)
- Sample mapping file with subject/infant ID, comparison variable and covariates (sam.rm; variables such as personid, age.sample, bf, gender)
- Group assignment vector (real, e.g. gender/breastfeeding status, or artificial case/control labels for null simulations)
- Design flags: comparison variable (comvar), covariates (adjustvar), longitudinal indicator

## Outputs

- Per-taxon comparison table from taxa.compare with estimates (log odds ratios of relative abundance), standard errors, p-values and FDR-adjusted p-values (taxacomtab, e.g. taxacom6.zi.rm.sex.adjustbfage)
- Per-iteration empirical type I error rates (proportion of taxa with p < 0.05 under null splits)
- Summary table of median (IQR) type I error per method (GAMLSS-BEZI vs LMAS) and taxonomic level (l2, l5), e.g. written to type1_error_table.csv matching Table 2

## How to apply

Load the processed relative abundance taxonomy table (e.g. taxtab.rm7 from the metamicrobiomeR package built on Subramanian et al. 2014 Bangladesh 16S data) and the sample map (sam.rm), optionally pre-filtering taxa with taxa.filter (percent.filter=0.05, relabund.filter=0.00005). Run taxa.compare with propmed.rel='gamlss' so GAMLSS-BEZI models the zero-inflated beta distribution of relative abundances directly, setting longitudinal='yes' so repeated samples per infant are handled, specifying the comparison variable (comvar) and covariates (adjustvar, e.g. age.sample or bf) and p.adjust.method='fdr'. In parallel fit linear models or linear mixed models with random intercepts per subject (LMAS / MaAsLin-style arcsin square root transformation) as a comparison method. Assess null-hypothesis behavior by permuting group labels over many iterations (e.g. 1000 random case/control splits preserving subject structure) and computing the proportion of taxa with p < alpha (0.01, 0.05, 0.1), summarizing empirical type I error as median (IQR) — well-controlled methods should stay close to nominal alpha at phylum (l2) and family (l5) levels.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare with propmed.rel='gamlss' (GAMLSS zero-inflated beta family) and longitudinal='yes' for differential relative abundance testing in longitudinal designs; install via devtools::install_github('nhanhocu/metamicrobiomeR')) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Statistical model implemented by metamicrobiomeR that directly addresses the zero-inflated beta distribution of microbiome relative abundance data; estimates are log(odds ratio) of relative abundances) — https://github.com/nhanhocu/metamicrobiomeR
- **MaAsLin** (Comparison method representing the linear model approach on arcsin square root transformed relative abundances (LMAS) used in parallel to benchmark type I error control)
- **knitr** (Used with kable() to render taxa comparison tables (taxcomtab.show) in the reproducible workflow)
- **R** (Programming language; requires R 3.4.2 or higher per package documentation) — https://github.com/nhanhocu/metamicrobiomeR

## Examples

```
taxacom<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="group",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- Under repeated null permutations (e.g. 1000 random case/control splits), median empirical type I error at phylum (l2) and family (l5) levels should fall close to the nominal alpha levels 0.01, 0.05 and 0.1 for both GAMLSS-BEZI and LMAS (as in Table 2 of the paper)
- Output table from taxa.compare contains per-taxon estimates interpretable as log(odds ratio) of relative abundances with FDR-adjusted p-values
- Longitudinal runs (longitudinal='yes') retain all repeated samples per subject rather than one sample per infant
- Filtered taxon list from taxa.filter (percent.filter=0.05, relabund.filter=0.00005) excludes sparse taxa before testing
- Results reproduce the published workflow when run on the Bangladesh cross-sectional (50-sample) and longitudinal (996-sample) subsets from the Gordon lab processed 16S data

## Limitations

- Linear/linear mixed model approaches (t-tests, Wilcoxon, MaAsLin-style arcsin square root transformation) do not address the zero-inflated beta distribution and zero inflation of microbiome relative abundance data
- Type I error control was demonstrated specifically on the Bangladesh cohort at phylum and family levels; behavior at deeper levels or other datasets should be re-verified
- The CRAN version of metamicrobiomeR has updates/minor changes relative to the initial GitHub version; users are recommended to use the CRAN version, which may alter exact reproduction
- Longitudinal GAMLSS comparisons can be computationally slow to run across many taxa and permutations

## Evidence

- [methods] GAMLSS with longitudinal design: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] Filtering thresholds: "taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005)"
- [intro] Zero-inflated beta rationale: "these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [discussion] GAMLSS-BEZI advantage: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [readme] Package install: "install_github("nhanhocu/metamicrobiomeR")"
- [abstract] Real-data performance: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
