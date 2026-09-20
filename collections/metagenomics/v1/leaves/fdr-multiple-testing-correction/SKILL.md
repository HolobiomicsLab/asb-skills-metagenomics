---
name: fdr-multiple-testing-correction
description: Use when testing differential relative abundances of many taxa (phylum
  to genus level, e.g. l2/l5/l6 tables) or pooling taxa estimates across studies,
  and raw p-values must be FDR-adjusted before calling significant taxa.
when_to_use_negative:
- Only a single pre-specified hypothesis test is run (no multiplicity across taxa),
  so correction is unnecessary
- The comparison table has fewer taxa than needed for FDR to be meaningful after taxa.filter
  (percent.filter=0.05, relabund.filter=0.00005) removed rare taxa, and the practitioner
  wants family-wise rather than FDR error control
- P-values come from heterogeneous, non-comparable tests across studies that were
  not pooled via meta.taxa with a common id and study label, so readjustment across
  them is invalid
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare, taxcomtab.show, meta.taxa and metatab.show, all of
    which accept p.adjust.method="fdr" and readjust.p to apply FDR correction to differential
    abundance and meta-analysis p-values
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Runtime language (R 3.4.2 or higher) in which the FDR-adjusted workflow is
    executed
- name: knitr
  role: Renders FDR-adjusted result tables (taxcomtab.show / metatab.show output)
    as kable tables
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:5d28b30d51d54cf2168cabbaf44307e8013a79eefd4f30d679f451adffc5ec8c
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fdr-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg false discovery rate (FDR) correction to p-values from GAMLSS-BEZI differential abundance tests and random-effects meta-analysis of microbiome relative abundances, both within a single study comparison (via taxa.compare) and as a re-adjustment on displayed result tables (taxcomtab.show, metatab.show) and pooled meta-analysis results (meta.taxa).

## When to use

Use when testing differential relative abundances of many taxa (phylum to genus level, e.g. l2/l5/l6 tables) or pooling taxa estimates across studies, and raw p-values must be FDR-adjusted before calling significant taxa.

## When NOT to use

- Only a single pre-specified hypothesis test is run (no multiplicity across taxa), so correction is unnecessary
- The comparison table has fewer taxa than needed for FDR to be meaningful after taxa.filter (percent.filter=0.05, relabund.filter=0.00005) removed rare taxa, and the practitioner wants family-wise rather than FDR error control
- P-values come from heterogeneous, non-comparable tests across studies that were not pooled via meta.taxa with a common id and study label, so readjustment across them is invalid

## Inputs

- taxa comparison result table from taxa.compare (per-taxon estimate, standard error, p-value; e.g. taxacom6.zi.rmg from GAMLSS-BEZI runs)
- combined multi-study taxa comparison table (e.g. tabsex4 built with plyr::rbind.fill from taxacom.rm.sex.adjustbfage, taxacom.ha.sex.adjustbfage, taxacom6.zi.usbmk.sex.adjustbfage, taxacom6.unc.sex.adjustedbfage)
- pooled random-effects meta-analysis table (metab.sex$random from meta.taxa)

## Outputs

- FDR-adjusted p-value columns appended to taxa comparison tables (readjusted within taxcomtab.show when readjust.p=TRUE)
- FDR-adjusted pooled p-values (p.adjust) in meta-analysis tables from meta.taxa
- Filtered result tables of significant taxa (e.g. phylum-level bfNon_exclusiveBF contrast, genus-level Coprococcus in males) exported as CSV or rendered via knitr::kable

## How to apply

Pass p.adjust.method="fdr" to taxa.compare so p-values for all taxa in a level (e.g. tax.lev="l2" phylum) are FDR-corrected within the GAMLSS-BEZI comparison. When displaying results with taxcomtab.show, set readjust.p=TRUE and p.adjust.method="fdr" to re-adjust p-values across the shown taxon set, optionally using p.cutoff=1 to display all taxa with adjusted values in the table. In cross-study meta-analysis, pass p.adjust.method="fdr" to meta.taxa so pooled random-effects p-values (summary.measure="RR", percent.meta=0.5) are FDR-corrected, then filter with metatab.show at p.cutoff=0.05 using p.cutoff.type="p" or the adjusted p. The rationale is that tens to hundreds of taxa are tested simultaneously, so uncorrected p-values would inflate false positives; FDR control was used throughout the Bangladesh breastfeeding, solid-food-timing, diarrhea-stratified, and four-study gender meta-analysis examples.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare, taxcomtab.show, meta.taxa and metatab.show, all of which accept p.adjust.method="fdr" and readjust.p to apply FDR correction to differential abundance and meta-analysis p-values) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Runtime language (R 3.4.2 or higher) in which the FDR-adjusted workflow is executed)
- **knitr** (Renders FDR-adjusted result tables (taxcomtab.show / metatab.show output) as kable tables)

## Examples

```
taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- All p-values in the output table fall within [0,1] and the adjusted p-value column reflects FDR readjustment applied across the displayed taxa (readjust.p=TRUE)
- For the Bangladesh non-EBF vs EBF phylum comparison, FDR-adjusted results show significant differences in all four observed phyla (Actinobacteria, Bacteroidetes, Firmicutes, Proteobacteria) under GAMLSS-BEZI
- For the solid-food-timing comparison, FDR-adjusted significance matches Table 4: Actinobacteria higher and Bacteroidetes and Firmicutes lower in the after-5-months group
- For the meta-analysis, no phylum remains significant after FDR adjustment, while genus Coprococcus retains FDR adjusted pooled p-value < 0.0001 for higher abundance in male infants
- metatab.show with p.cutoff=0.05 (p.cutoff.type="p") returns only rows whose FDR-adjusted pooled p-values pass the cutoff

## Limitations

- FDR readjustment in taxcomtab.show depends on which taxa subset is displayed (tax.select), so readjusted p-values can differ from the original taxa.compare adjustments if the taxon set changes
- The GitHub package version has minor changes relative to the recommended CRAN version, which users are advised to use instead
- The GitHub version itself does not ship inline datasets/accessions documentation; the tutorial (Additional file 1) must be consulted for full reproducibility

## Evidence

- [readme] taxa.compare with FDR in README example: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [readme] readjust.p in taxcomtab.show: "taxcomtab.show(taxcomtab=taxacom6.zi.rmg,tax.select=p.bf.l2$taxuse.rm, showvar="bfNon_exclusiveBF", tax.lev="l2",readjust.p=TRUE,p.adjust.method="fdr",p.cutoff = 1)"
- [methods] meta.taxa with FDR correction: "metab.sex<-meta.taxa(taxcomdat=tabsex4,summary.measure="RR",pool.var="id",studylab="study",backtransform=FALSE,percent.meta=0.5,p.adjust.method="fdr")"
- [readme] metatab.show FDR-filtered display: "metatab.show(metatab=metab.sex$random,com.pooled.tab=tabsex4,tax.lev="l2",showvar="genderMale",p.cutoff.type="p", p.cutoff=0.05,display="table")"
