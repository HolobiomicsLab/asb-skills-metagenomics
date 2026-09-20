---
name: longitudinal-covariate-adjustment
description: Use when comparing microbiome taxon relative abundances between groups
  using longitudinal cohort data (multiple samples per infant, e.g. Bangladesh cohort
  ≤6 months), where a covariate such as infant age at sample collection must be adjusted
  for in the model.
when_to_use_negative:
- Data are cross-sectional with a single sample per subject (longitudinal='yes' and
  repeated-measure handling are unnecessary)
- Input is count data or absolute abundance data rather than zero-inflated relative
  abundance (GAMLSS-BEZI models relative abundances)
- The goal is meta-analysis pooling across studies rather than within-study longitudinal
  covariate-adjusted comparison
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare (GAMLSS-BEZI fitting with longitudinal and covariate
    adjustment), taxa.meansdn, taxa.mean.plot, taxa.filter, and taxcomtab.show for
    the differential abundance workflow
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Programming language in which the package and workflow run (R 3.4.2 or higher)
- name: devtools
  role: Used to install the package from GitHub via install_github('nhanhocu/metamicrobiomeR')
- name: knitr
  role: Renders the result tables with knitr::kable
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:39504e780e18bdee1eafbe58f0c04d64516ec20d93eacb8146b823b9af35bc1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - devtools
  - gdata
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# longitudinal-covariate-adjustment

## Summary

A differential abundance testing pattern that models repeated stool samples per infant with GAMLSS-BEZI (zero-inflated beta) while adjusting for a longitudinal covariate — here, infant age at sample collection (age.sample) — so that group contrasts (e.g. non-exclusive vs exclusive breastfeeding, comvar='bf') reflect the covariate-adjusted log(odds ratio) of relative abundance.

## When to use

Use when comparing microbiome taxon relative abundances between groups using longitudinal cohort data (multiple samples per infant, e.g. Bangladesh cohort ≤6 months), where a covariate such as infant age at sample collection must be adjusted for in the model.

## When NOT to use

- Data are cross-sectional with a single sample per subject (longitudinal='yes' and repeated-measure handling are unnecessary)
- Input is count data or absolute abundance data rather than zero-inflated relative abundance (GAMLSS-BEZI models relative abundances)
- The goal is meta-analysis pooling across studies rather than within-study longitudinal covariate-adjusted comparison

## Inputs

- Bundled Bangladesh taxa relative abundance table data(taxtab.rm7) (QIIME-derived phylum-level table, list element [[5]])
- Sample mapping file variables: comvar='bf' (breastfeeding status: ExclusiveBF / Non_exclusiveBF), adjustvar='age.sample' (infant age in months at stool sample collection), personid/sampleid for repeated measures
- Longitudinal metadata for infants ≤6 months of age

## Outputs

- taxa.compare result table with estimates (log odds ratio), 95% CI, and FDR-adjusted p-values per taxon (e.g. taxacom6.zi.rmg)
- taxcomtab.show output table for the bfNon_exclusiveBF contrast at phylum level (l2), exportable as e.g. bf_phylum_comparison.csv
- Optional mean relative abundance plots by breastfeeding status and age (taxa.mean.plot, p.bf.l2$p)

## How to apply

Load the metamicrobiomeR package and the bundled Bangladesh taxa table with data(taxtab.rm7); restrict to infants ≤6 months and to the breastfed groups (bf = ExclusiveBF / Non_exclusiveBF), dropping the No_BF group. Run taxa.compare on the phylum-level taxa table with propmed.rel='gamlss' (zero-inflated beta GAMLSS-BEZI, which directly addresses the zero-inflated beta distribution of relative abundances), comvar='bf' as the comparison variable, adjustvar='age.sample' as the covariate to adjust for, longitudinal='yes' to account for repeated measures on the same infants, and p.adjust.method='fdr'. Display the contrast of interest (showvar='bfNon_exclusiveBF') at taxonomic level l2 with taxcomtab.show, setting readjust.p=TRUE with FDR readjustment; estimates are log(odds ratio) of relative abundance between the comparison groups. Optionally first summarize and visualize mean relative abundance by bf and age.sample with taxa.meansdn and taxa.mean.plot (tax.lev='l2', mean.filter=0.005, show.taxname='short') and filter taxa with taxa.filter (percent.filter=0.05, relabund.filter=0.00005).

## Related tools

- **metamicrobiomeR** (Provides taxa.compare (GAMLSS-BEZI fitting with longitudinal and covariate adjustment), taxa.meansdn, taxa.mean.plot, taxa.filter, and taxcomtab.show for the differential abundance workflow) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Programming language in which the package and workflow run (R 3.4.2 or higher))
- **devtools** (Used to install the package from GitHub via install_github('nhanhocu/metamicrobiomeR'))
- **knitr** (Renders the result tables with knitr::kable)

## Examples

```
taxacom6.zi.rmg <- taxa.compare(taxtab=taxtab6.rm[[5]], propmed.rel="gamlss", comvar="bf", adjustvar="age.sample", longitudinal="yes", p.adjust.method="fdr")
```

## Evaluation signals

- Output table exists and contains rows for the phylum-level taxa (l2) with numeric estimate, 95% CI, and FDR-adjusted p-value columns
- All p-values fall within [0,1] and FDR readjustment was applied (readjust.p=TRUE, p.adjust.method='fdr')
- All four observed phyla (Actinobacteria, Bacteroidetes, Firmicutes, Proteobacteria) yield results for the bfNon_exclusiveBF contrast; significant differences should be detected in all four, whereas LMAS detects only three
- The model respects repeated measures: longitudinal='yes' was set and infant age (age.sample) was included as adjustvar
- Estimates are interpretable as log(odds ratio) of relative abundance between non-exclusive and exclusive breastfeeding groups

## Limitations

- GAMLSS-BEZI modeling can be computationally slow for taxa-rich tables (the related microbiomeage example notes 'take time to run'); phylum-level comparisons are fast, deeper levels take longer
- Requires repeated-measure/longitudinal metadata (person/sample identifiers and age.sample); results depend on correct restriction to infants ≤6 months and removal of the No_BF group before plotting/selection
- Estimates are log(odds ratio) of relative abundance — analogous across studies but not absolute abundance effect sizes

## Evidence

- [methods] taxa.compare with longitudinal adjustment: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] FDR readjustment of shown table: "taxcomtab.show(taxcomtab=taxacom6.zi.rmg,tax.select=p.bf.l2$taxuse.rm, showvar="bfNon_exclusiveBF", tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr",p.cutoff = 1)"
- [abstract] GAMLSS-BEZI rationale: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [abstract] estimate interpretation: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [readme] readme example invocation: "Comparison of bacterial taxa relative abundance using GAMLSS"
- [intro] distribution rationale: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
