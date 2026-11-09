---
name: zero-inflated-beta-gamlss-modeling
description: Use when testing differential relative abundance of microbial taxa between
  comparison groups on compositional, zero-inflated proportion data (e.g., case/control
  taxa tables), where t-tests or transformed linear models would mishandle excess
  zeros.
when_to_use_negative:
- Input is raw count or OTU/ASV feature-table data not yet summarized to relative
  abundance — the BEZI model targets relative abundance proportions with zero inflation
- The goal is alpha diversity comparison or microbiome age prediction, which use alpha.compare
  or microbiomeage functions instead of taxa.compare/GAMLSS-BEZI
- Taxa have no zeros and near-normal abundance distributions where simpler linear
  models suffice, or the data are not compositional proportions
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: R package providing taxa.compare (propmed.rel='gamlss') to fit GAMLSS-BEZI
    models and taxa.filter/taxcomtab.show for pre-filtering and result display
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Statistical model (zero-inflated beta family within GAMLSS) that directly
    addresses the distribution of microbiome relative abundance data
- name: R
  role: Implementation language; R 3.4.2 or higher required
- name: ggplot2
  role: Used for plotting mean relative abundance and ROC curve visualizations in
    the workflow
- name: LMAS
  role: Benchmark comparison method whose differential abundance p-values are contrasted
    against GAMLSS-BEZI via ROC/AUC and DeLong's test
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:4c1a213d3bc69a77b5fbdab4a3ba55fc3c998899a7514c49525d5955c64ae58c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - LMAS
  - R
  - ggplot2
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# zero-inflated-beta-gamlss-modeling

## Summary

Fit Generalized Additive Models for Location, Scale and Shape (GAMLSS) with a zero-inflated beta (BEZI) family to microbiome relative abundance data, via metamicrobiomeR's taxa.compare, to test differential relative abundance between groups while directly modeling zero inflation and the beta distribution of proportions.

## When to use

Use when testing differential relative abundance of microbial taxa between comparison groups on compositional, zero-inflated proportion data (e.g., case/control taxa tables), where t-tests or transformed linear models would mishandle excess zeros.

## When NOT to use

- Input is raw count or OTU/ASV feature-table data not yet summarized to relative abundance — the BEZI model targets relative abundance proportions with zero inflation
- The goal is alpha diversity comparison or microbiome age prediction, which use alpha.compare or microbiomeage functions instead of taxa.compare/GAMLSS-BEZI
- Taxa have no zeros and near-normal abundance distributions where simpler linear models suffice, or the data are not compositional proportions

## Inputs

- Taxon-by-sample relative abundance table (e.g., QIIME-derived taxonomic summary up to phylum/genus level, taxtab levels l2-l6)
- Sample mapping/metadata file with comparison variable (comvar), covariates (adjustvar), and sample IDs (e.g., bf, gender, age.sample)
- Simulated zero-inflated beta relative abundance data with known ground-truth differential/null taxa (for benchmarking)

## Outputs

- Per-taxon differential abundance comparison table (taxa.compare output) with estimates (log odds ratio), p-values, and FDR-adjusted p-values
- Filtered taxon list from taxa.filter (e.g., taxlist.rm at percent.filter=0.05, relabund.filter=0.00005)
- Display-ready tables via taxcomtab.show and ROC/AUC benchmark records (AUC with 95% CI, DeLong p-value) when used in simulations

## How to apply

Install metamicrobiomeR from GitHub (nhanhocu/metamicrobiomeR; R 3.4.2+ required) or CRAN, and first apply taxa.filter with percent.filter=0.05 and relabund.filter=0.00005 to retain taxa with sufficient prevalence/abundance. Then call taxa.compare with propmed.rel='gamlss' so GAMLSS with the BEZI family is fitted to each taxon's relative abundance, specifying the comparison variable (comvar, e.g. 'gender' or 'bf'), covariates (adjustvar, e.g. 'age.sample'), longitudinal='yes' for repeated measures, and p.adjust.method='fdr' for multiple testing. Rationale: microbiome relative abundances resemble a zero-inflated beta distribution that standard LM/LMM approaches (e.g., arcsin square-root transformation as in MaAsLin) do not address; BEZI-GAMLSS models it directly. Interpret the returned estimates as log(odds ratio) of relative abundances between groups, which makes them analogous across studies and poolable via random-effects meta-analysis (meta.taxa with summary.measure='RR'). For simulation benchmarks, feed the p-values into pROC to compute ROC/AUC and use DeLong's test for paired AUC comparison (GAMLSS-BEZI AUC 95.6, CI 94.2-97.1 vs LMAS 92.9, CI 91.1-94.7; p < 2.2e-16 in the paper's 800-species simulation with n=100 per group).

## Related tools

- **metamicrobiomeR** (R package providing taxa.compare (propmed.rel='gamlss') to fit GAMLSS-BEZI models and taxa.filter/taxcomtab.show for pre-filtering and result display) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Statistical model (zero-inflated beta family within GAMLSS) that directly addresses the distribution of microbiome relative abundance data)
- **R** (Implementation language; R 3.4.2 or higher required)
- **ggplot2** (Used for plotting mean relative abundance and ROC curve visualizations in the workflow)
- **LMAS** (Benchmark comparison method whose differential abundance p-values are contrasted against GAMLSS-BEZI via ROC/AUC and DeLong's test)

## Examples

```
taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- In the 800-species simulation (400 null, 400 differential across four mu-shift settings, n=100 per group), reproduced results should show GAMLSS-BEZI AUC = 95.6 (95% CI 94.2-97.1) versus LMAS AUC = 92.9 (95% CI 91.1-94.7) with DeLong p < 2.2e-16
- taxa.compare is called with propmed.rel='gamlss' and p.adjust.method='fdr'; output table must contain FDR-adjusted p-values per taxon
- Estimates are interpretable as log(odds ratio) of relative abundances between comparison groups — check sign and magnitude consistency with group means
- Upstream taxa.filter with percent.filter=0.05 and relabund.filter=0.00005 should have pruned low-prevalence/low-abundance taxa before model fitting
- Repeated-measures designs must set longitudinal='yes' in taxa.compare so within-subject correlation is handled

## Limitations

- t-tests, Wilcoxon tests, and linear/linear mixed models do not address the zero-inflated beta distribution of relative abundance data — results from such approaches are not interchangeable with GAMLSS-BEZI fits
- Arcsin square root transformation (as in MaAsLin) does not address the inflation of zero values in relative abundance data
- The GitHub version has minor changes compared to the CRAN version; users are recommended to use the CRAN version
- Requires R 3.4.2 or higher; the microbiomeage/Random Forest components are time-consuming to run (though separate from taxa.compare)

## Evidence

- [abstract] We developed an R package 'metamicrobiomeR' that applies Generalized Additive Models for Location, Scale and Shape (GAMLSS) with a zero-inflated beta (BEZI) family: "We developed an R package 'metamicrobiomeR' that applies Generalized Additive Models for Location, Scale and Shape (GAMLSS) with a zero-inflated beta (BEZI) family"
- [intro] these methods do not address the actual distribution of the microbial taxonomy relative abundance data: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [methods] taxa.compare example call from the methods section: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005): "taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005)"
- [abstract] the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [readme] Users are recommended to use the R cran version.: "There are some updates/ minor changes in the R cran version as compared to this initial version of the package deposited in Github.  Users are recommended to use the R cran version."
