---
name: longitudinal-microbiome-taxa-comparison
description: Use when testing differential taxa relative abundances (phylum to genus,
  taxtab levels l2–l6) between two groups in longitudinal/repeated-measures microbiome
  relative abundance data with many zeros.
when_to_use_negative:
- Data are count-based feature tables rather than relative abundances — the zero-inflated
  beta model assumes relative abundance inputs
- The comparison variable varies within subjects but you cannot supply a person ID,
  since longitudinal='yes' requires repeated-measures structure
- The goal is pooling estimates across multiple studies — use meta.taxa random-effects
  meta-analysis on taxa.compare outputs instead
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare/taxcomtab.show implementing GAMLSS-BEZI differential
    abundance testing for relative abundance data with longitudinal support
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Runtime environment (R 3.4.2 or higher) in which metamicrobiomeR is installed
    and the taxa.compare workflow is executed
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: gamlss
  role: Zero-inflated beta (BEZI) GAMLSS modeling framework underlying propmed.rel='gamlss'
    in taxa.compare
- name: knitr
  role: Renders the taxcomtab.show comparison table via kable for display/export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:79ecc809c97ffdaa11f01a58a7e6c6e13a48fd3cf9ddea1b76a37a38fd270768
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
yes_rate: 0.7
n_claims: 10
---

# longitudinal-microbiome-taxa-comparison

## Summary

Fits zero-inflated beta GAMLSS-BEZI models (via metamicrobiomeR::taxa.compare) to test differential bacterial taxa relative abundances between groups in repeated-measures infant microbiome data, adjusting for age and handling the zero-inflated beta distribution that linear models like MaAsLin miss.

## When to use

Use when testing differential taxa relative abundances (phylum to genus, taxtab levels l2–l6) between two groups in longitudinal/repeated-measures microbiome relative abundance data with many zeros.

## When NOT to use

- Data are count-based feature tables rather than relative abundances — the zero-inflated beta model assumes relative abundance inputs
- The comparison variable varies within subjects but you cannot supply a person ID, since longitudinal='yes' requires repeated-measures structure
- The goal is pooling estimates across multiple studies — use meta.taxa random-effects meta-analysis on taxa.compare outputs instead

## Inputs

- Taxa relative abundance table (e.g. taxtab.rm7, taxonomic level 5) with samples as columns and taxa up to genus level as rows
- Sample mapping/metadata with a binary comparison variable (e.g. bf, gender, solid-food timing group), an age variable (age.sample), and a repeated-measures person ID
- Optional taxa list from taxa.mean.plot/taxa.filter (taxuse.rm) for selecting taxa to display

## Outputs

- taxa.compare result object of per-taxon GAMLSS-BEZI estimates (log odds ratios), standard errors, raw and FDR-adjusted p-values
- Formatted comparison table via taxcomtab.show (renderable with knitr::kable) at a chosen taxonomic level (l2–l6)

## How to apply

Load a taxa relative abundance table (e.g. the Bangladesh cohort taxtab.rm7 at taxonomic level 5) and filter with taxa.filter (percent.filter=0.05, relabund.filter=0.00005). Run taxa.compare with propmed.rel='gamlss' to invoke the zero-inflated beta GAMLSS-BEZI model, set comvar to the binary comparison variable (e.g. timing of solid-food introduction after vs before 5 months, or breastfeeding status), adjustvar to covariates such as age.sample, and longitudinal='yes' so the repeated-measures person ID is respected. Restrict the analysis window (e.g. age.sample between 6 months and 2 years) before fitting. Summarize with taxcomtab.show at the chosen taxonomic level (tax.lev='l2' for phylum) and re-adjust p-values with p.adjust.method='fdr'; GAMLSS-BEZI estimates are log(odds ratio) of relative abundances, which keeps them analogous across studies.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare/taxcomtab.show implementing GAMLSS-BEZI differential abundance testing for relative abundance data with longitudinal support) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Runtime environment (R 3.4.2 or higher) in which metamicrobiomeR is installed and the taxa.compare workflow is executed) — https://github.com/nhanhocu/metamicrobiomeR
- **gamlss** (Zero-inflated beta (BEZI) GAMLSS modeling framework underlying propmed.rel='gamlss' in taxa.compare)
- **knitr** (Renders the taxcomtab.show comparison table via kable for display/export)

## Examples

```
taxacom6.zi.rmg <- taxa.compare(taxtab=taxtab6.rm[[5]], propmed.rel="gamlss", comvar="bf", adjustvar="age.sample", longitudinal="yes", p.adjust.method="fdr")
```

## Evaluation signals

- FDR-adjusted p-values in the taxcomtab.show table reproduce the paper's significant phyla: for the solid-food-timing example, higher Actinobacteria and lower Bacteroidetes and Firmicutes in the after-5-months group (all three detected, vs only one under MaAsLin-style linear modeling)
- Output table columns include estimates (log odds ratios), standard errors, raw p-values and FDR-adjusted p-values per taxon at the requested tax.lev
- taxa.compare was called with propmed.rel='gamlss', longitudinal='yes' and p.adjust.method='fdr' — verify these parameters in the call
- Taxa displayed match the filtered list (e.g. tax.use from taxa.filter with percent.filter=0.05, relabund.filter=0.00005) or tax.select='none' shows all tested taxa
- Estimates are interpretable as log(odds ratio) of relative abundances between comparison groups

## Limitations

- Linear/linear mixed models (e.g. MaAsLin with arcsin square root transformation) do not address the inflation of zero values and detected fewer significant phyla than GAMLSS-BEZI on the same data
- The CRAN version of metamicrobiomeR has minor updates/changes relative to the initial GitHub version; users are recommended to use the CRAN version
- Results depend on the taxa.filter thresholds (percent.filter=0.05, relabund.filter=0.00005) and any age-window restriction applied before testing

## Evidence

- [methods] taxa.compare call with GAMLSS-BEZI and FDR: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [abstract] GAMLSS-BEZI performance on differential abundance testing: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [readme] Package implements GAMLSS with BEZI family: "The *metamicrobiomeR* package implements Generalized Additive Model for Location, Scale and Shape (GAMLSS) with zero inflated beta (BEZI) family for analysis of microbiome relative abundance data"
- [intro] Limitation of linear-model approaches: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
