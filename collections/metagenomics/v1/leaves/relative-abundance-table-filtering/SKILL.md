---
name: relative-abundance-table-filtering
description: Use when a relative abundance table (taxonomy levels l2–l6, e.g. Bangladesh
  taxtab.rm[[5]]) contains rare taxa that would destabilize downstream GAMLSS-BEZI
  differential abundance testing or clutter phylum-level mean plots.
when_to_use_negative:
- Input is an absolute-count (OTU/feature) table rather than a relative abundance
  table — normalize to relative abundances first
- Downstream analysis is meta-analysis pooling (meta.taxa) of already-computed taxa.compare
  estimates — filtering should have been applied upstream per study
- The table has already been filtered with the same percent.filter/relabund.filter
  thresholds and no new rare taxa were introduced
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: Provides taxa.filter, taxa.meansdn and taxa.mean.plot used to filter and summarize
    relative abundance tables before GAMLSS-BEZI comparison
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Runtime environment; metamicrobiomeR requires R 3.4.2 or higher
- name: knitr
  role: Renders the filtered taxa comparison/mean tables via kable for display
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:f84ac2efd48e1da002f1df9713bdfd3ebdc970b6632db534114abd8cd90e3fca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# relative-abundance-table-filtering

## Summary

A pre-analysis filtering skill from metamicrobiomeR that removes rare bacterial taxa from taxonomy-level relative abundance tables (e.g., taxtab.rm7 level 5 = genus) using prevalence and abundance thresholds, plus optional mean-abundance filtering for plotting and selecting taxa to display downstream.

## When to use

Use when a relative abundance table (taxonomy levels l2–l6, e.g. Bangladesh taxtab.rm[[5]]) contains rare taxa that would destabilize downstream GAMLSS-BEZI differential abundance testing or clutter phylum-level mean plots.

## When NOT to use

- Input is an absolute-count (OTU/feature) table rather than a relative abundance table — normalize to relative abundances first
- Downstream analysis is meta-analysis pooling (meta.taxa) of already-computed taxa.compare estimates — filtering should have been applied upstream per study
- The table has already been filtered with the same percent.filter/relabund.filter thresholds and no new rare taxa were introduced

## Inputs

- Taxonomy-level relative abundance table (e.g., data(taxtab.rm7), taxtab.rm[[5]] for genus-level Bangladesh data)
- Grouping/mapping variables in sample metadata (e.g., bf, age.sample, gender) for taxa.meansdn/taxa.mean.plot
- Threshold parameters: percent.filter (e.g., 0.05), relabund.filter (e.g., 0.00005), mean.filter (e.g., 0.005)

## Outputs

- Filtered taxa list (taxlist.rm) used by taxa.compare
- Mean/SD summary table per group and age group (taxa.meansdn.rm)
- Plot object of mean relative abundance by group and age (p.bf.l2$p) and taxuse.rm taxon-selection list

## How to apply

Apply taxa.filter to the desired taxonomic level of the relative abundance table (e.g., taxtab.rm[[5]] for genus) with percent.filter=0.05 (taxon present in at least 5% of samples) and relabund.filter=0.00005 (minimum relative abundance ~0.005%), returning taxlist.rm — the list of taxa to retain for taxa.compare. For display purposes, first summarize means/SDs per group with taxa.meansdn (sumvar e.g. 'bf', groupvar 'age.sample'), optionally subsetting (e.g., bf != 'No_BF' and age.sample <= 6 months), then use taxa.mean.plot with mean.filter=0.005 and tax.lev='l2' to keep taxa above a mean relative abundance of 0.5%; the resulting p.bf.l2$taxuse.rm list feeds taxcomtab.show so only well-supported phyla/genera appear in comparison tables. Rationale: rare taxa inflate zeros and produce unstable zero-inflated beta GAMLSS fits, so filtering concentrates testing on taxa with adequate support.

## Related tools

- **metamicrobiomeR** (Provides taxa.filter, taxa.meansdn and taxa.mean.plot used to filter and summarize relative abundance tables before GAMLSS-BEZI comparison) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Runtime environment; metamicrobiomeR requires R 3.4.2 or higher)
- **knitr** (Renders the filtered taxa comparison/mean tables via kable for display)

## Examples

```
library(metamicrobiomeR); data(taxtab.rm7); taxlist.rm <- taxa.filter(taxtab=taxtab.rm[[5]], percent.filter=0.05, relabund.filter=0.00005)
```

## Evaluation signals

- taxa.filter returns a reduced taxon list (taxlist.rm) strictly smaller than the input table columns, retaining only taxa meeting percent.filter=0.05 and relabund.filter=0.00005
- taxa.mean.plot with mean.filter=0.005 excludes phyla with mean relative abundance below 0.005; p.bf.l2$taxuse.rm contains only surviving phylum-level taxa
- Downstream taxcomtab.show tables list only taxa in taxuse.rm, and FDR-adjusted significant phyla (e.g., Actinobacteria higher, Bacteroidetes/Firmicutes lower) reproduce Table 4 of the paper
- All values in the filtered table remain in [0,1] and each sample's relative abundances remain interpretable as proportions

## Limitations

- Thresholds (percent.filter=0.05, relabund.filter=0.00005, mean.filter=0.005) are example defaults from the Bangladesh cohort; other studies may require retuning
- Filtering removes taxa from all subsequent differential testing, so genuinely low-prevalence differentially abundant taxa cannot be detected
- The GitHub version differs from the CRAN version of metamicrobiomeR, which the authors recommend; minor function behavior may vary

## Evidence

- [methods] taxa.filter applied with prevalence and abundance thresholds: "taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005)"
- [readme] mean/SD summarization by group and age before plotting: "taxa.meansdn.rm<-taxa.meansdn(taxtab=taxtab.rm[[5]],sumvar="bf",groupvar="age.sample")"
- [readme] phylum-level plot with mean-abundance filter: "p.bf.l2<-taxa.mean.plot(tabmean=taxa.meansdn.rm,tax.lev="l2", comvar="bf", groupvar="age.sample",mean.filter=0.005, show.taxname="short")"
- [intro] motivation: relative abundances follow zero-inflated beta distribution: "these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
