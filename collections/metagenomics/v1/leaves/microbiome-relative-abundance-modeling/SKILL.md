---
name: microbiome-relative-abundance-modeling
description: Use when testing differential relative abundance of microbial taxa (phylum
  to genus, e.g. levels l2/l4/l5/l6) between groups in compositional, zero-inflated
  16S microbiome data, especially longitudinal designs with covariate adjustment.
when_to_use_negative:
- Data are not relative abundances (e.g. already a feature/count table without compositional
  normalization) — the BEZI family models zero-inflated beta relative abundance data
  specifically
- The goal is alpha diversity comparison — use alpha.compare with standardized diversity
  indexes and linear/linear mixed models instead
- Input is metagenomic functional profiles or non-taxon omics data — the workflow
  targets bacterial taxon relative abundance tables from 16S-style pipelines (e.g.
  QIIME L6 taxonomy tables)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: metamicrobiomeR
  role: Implements GAMLSS with zero-inflated beta (BEZI) family via taxa.compare for
    differential relative abundance testing; also provides taxa.filter, taxa.meansdn,
    taxa.mean.plot, and taxcomtab.show
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Programming language for the package; requires R 3.4.2 or higher
- name: devtools
  role: Used to install the GitHub version of metamicrobiomeR (install_github("nhanhocu/metamicrobiomeR"))
- name: knitr
  role: Renders result tables (kable) of taxa.compare/taxcomtab.show output
- name: MaAsLin
  role: Comparison method (LM with arcsin square root transformation) that does not
    address zero inflation; serves as the LMAS baseline the BEZI approach outperforms
    (AUC 95.6 vs 92.9, DeLong p < 2.2e-16)
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:cf3eef14196aeab87db016fc805fb81187e2fa85dad4282b98ed6cfa546ebdf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - LMAS
  - MaAsLin
  - R
  - devtools
  - gdata
  - ggplot2
  - knitr
  - metamicrobiomeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# microbiome-relative-abundance-modeling

## Summary

Model microbiome taxon relative abundances with GAMLSS using a zero-inflated beta (BEZI) family via metamicrobiomeR's taxa.compare, directly addressing the zero-inflated beta distribution of relative abundance data where linear/linear mixed models and arcsin-square-root transformations (e.g., MaAsLin/LMAS) fail.

## When to use

Use when testing differential relative abundance of microbial taxa (phylum to genus, e.g. levels l2/l4/l5/l6) between groups in compositional, zero-inflated 16S microbiome data, especially longitudinal designs with covariate adjustment.

## When NOT to use

- Data are not relative abundances (e.g. already a feature/count table without compositional normalization) — the BEZI family models zero-inflated beta relative abundance data specifically
- The goal is alpha diversity comparison — use alpha.compare with standardized diversity indexes and linear/linear mixed models instead
- Input is metagenomic functional profiles or non-taxon omics data — the workflow targets bacterial taxon relative abundance tables from 16S-style pipelines (e.g. QIIME L6 taxonomy tables)

## Inputs

- Taxon relative abundance table (e.g. taxtab.rm7 / taxtab.rm[[5]] from metamicrobiomeR, phylum-to-genus levels l2-l6)
- Sample mapping/metadata with comparison variable (e.g. bf with ExclusiveBF/Non_exclusiveBF/No_BF), covariates (age.sample), and subject ID for longitudinal structure
- Bangladesh cohort 16S data (Subramanian et al 2014) from gordonlab.wustl.edu processed 16S rRNA datasets

## Outputs

- taxa.compare result object with per-taxon estimates (log odds ratio), 95% CI, and FDR-adjusted p-values (e.g. taxacom6.zi.rmg)
- Formatted comparison tables via taxcomtab.show rendered with knitr::kable (e.g. bf_phylum_comparison.csv)
- Mean relative abundance summaries/plots via taxa.meansdn and taxa.mean.plot

## How to apply

Install metamicrobiomeR (devtools::install_github("nhanhocu/metamicrobiomeR") or the CRAN version recommended in the README) and load the taxa relative abundance table (e.g. data(taxtab.rm7), taking taxtab.rm[[5]] for a taxonomic level). Optionally pre-filter with taxa.filter(percent.filter=0.05, relabund.filter=0.00005) and summarize/plot mean relative abundance with taxa.meansdn and taxa.mean.plot (mean.filter=0.005, show.taxname="short"). Run taxa.compare with propmed.rel="gamlss" (the zero-inflated beta GAMLSS-BEZI family), specifying the comparison variable (comvar, e.g. "bf" or "gender"), covariates (adjustvar, e.g. "age.sample"), longitudinal="yes" for repeated samples per infant, and p.adjust.method="fdr". Display results with taxcomtab.show at the desired tax level (e.g. tax.lev="l2" for phylum, showvar="bfNon_exclusiveBF", readjust.p=TRUE, p.cutoff=1). Estimates are log(odds ratio) of relative abundances between comparison groups, making them analogous across studies and poolable via random-effects meta-analysis (meta.taxa).

## Related tools

- **metamicrobiomeR** (Implements GAMLSS with zero-inflated beta (BEZI) family via taxa.compare for differential relative abundance testing; also provides taxa.filter, taxa.meansdn, taxa.mean.plot, and taxcomtab.show) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Programming language for the package; requires R 3.4.2 or higher)
- **devtools** (Used to install the GitHub version of metamicrobiomeR (install_github("nhanhocu/metamicrobiomeR")))
- **knitr** (Renders result tables (kable) of taxa.compare/taxcomtab.show output)
- **MaAsLin** (Comparison method (LM with arcsin square root transformation) that does not address zero inflation; serves as the LMAS baseline the BEZI approach outperforms (AUC 95.6 vs 92.9, DeLong p < 2.2e-16))

## Examples

```
taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- Output table exists with rows for taxa at the requested level and numeric estimate, 95% CI, and FDR-adjusted p-value columns; all p-values in [0,1] with FDR readjustment applied
- Reproducing the breastfeeding comparison should detect significant differences in all four observed phyla (Actinobacteria, Bacteroidetes, Firmicutes, Proteobacteria), whereas LMAS detects only three
- Simulated-data benchmark: GAMLSS-BEZI AUC 95.6 (95% CI 94.2-97.1) vs LMAS 92.9 (91.1-94.7), DeLong p < 2.2e-16
- Real-data type I error: median empirical type I error at phylum and family levels close to nominal alpha of 0.01, 0.05, and 0.1 across 1000 random case/control splits
- Estimates are interpretable as log(odds ratio) of relative abundances; e.g. diarrhea in EBF<=2mo stratum shows Firmicutes estimate 0.49 (FDR p=0.0109) and Actinobacteria -0.73 (FDR p=0.0011)

## Limitations

- Linear/linear mixed models and arcsin square root transformation (MaAsLin/LMAS) do not address the zero-inflated beta distribution of relative abundance data — avoid substituting them for the BEZI fit
- The CRAN version has updates/minor changes relative to the initial GitHub version; users are recommended to use the CRAN version
- Requires R 3.4.2 or higher; longitudinal analysis requires repeated samples per infant identified via subject IDs
- The GitHub tutorial (Additional file 1) is referenced but not inline-accessible in the article text

## Evidence

- [discussion] GAMLSS-BEZI directly addresses distribution: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [abstract] BEZI models zero-inflated relative abundance: "applies Generalized Additive Models for Location, Scale and Shape (GAMLSS) with a zero-inflated beta (BEZI) family"
- [methods] taxa.compare call with gamlss parameters: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [readme] README taxa.compare example: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [intro] LM/asin-sqrt does not address zero inflation: "However, this adjustment does not address the inflation of zero values in microbiome relative abundance data."
- [abstract] Estimates are log odds ratios: "the estimates from GAMLSS-BEZI are log (odds ratio) of relative abundances between comparison groups and thus are analogous between microbiome studies"
- [discussion] R version requirement: "Programming language: R. Other requirements: R 3.4.2 or higher."
