---
name: heatmap-forest-plot-visualization
description: Use when pooled random-effects meta-analysis results (e.g. genderMale
  comparison of gut taxa across four studies) need to be visualized as side-by-side
  heatmaps of effect estimates and forest plots at phylum (level='main') or family/genus
  (level='sub') levels.
when_to_use_negative:
- When the goal is a publication table of significant taxa rather than a figure —
  use metatab.show with display='table' and p.cutoff=0.05 instead
- When input is raw taxa relative abundance tables rather than pooled meta-analysis
  output — taxa.mean.plot or taxa.compare outputs are not valid metadat inputs
- When visualizing alpha diversity or microbiome age results, which use their own
  summary objects rather than meta.taxa output
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: metamicrobiomeR
  role: provides metatab.show (display='data') and meta.niceplot functions used to
    build and render the heatmap-forest plots of pooled taxa meta-analysis results
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: runtime environment (R 3.4.2 or higher) in which the plotting workflow is
    executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:47ac72a1a2c22c38fc6513b7bdc264feff9d12067f35271c08d4e4ae6a3ebd71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - devtools
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# heatmap-forest-plot-visualization

## Summary

Render combined heatmap-forest plots of pooled meta-analysis results for microbial taxa relative abundances using metamicrobiomeR's meta.niceplot function, after preparing display data with metatab.show (display='data').

## When to use

Use when pooled random-effects meta-analysis results (e.g. genderMale comparison of gut taxa across four studies) need to be visualized as side-by-side heatmaps of effect estimates and forest plots at phylum (level='main') or family/genus (level='sub') levels.

## When NOT to use

- When the goal is a publication table of significant taxa rather than a figure — use metatab.show with display='table' and p.cutoff=0.05 instead
- When input is raw taxa relative abundance tables rather than pooled meta-analysis output — taxa.mean.plot or taxa.compare outputs are not valid metadat inputs
- When visualizing alpha diversity or microbiome age results, which use their own summary objects rather than meta.taxa output

## Inputs

- metatab object from meta.taxa$random (random-effects pooled results)
- combined per-study comparison table (e.g. tabsex4 built with plyr::rbind.fill from the four taxacom datasets)
- display data frame from metatab.show(display='data') at tax.lev 'l2', 'l5', or 'l6'

## Outputs

- heatmap-forest ggplot figure of pooled taxa effect estimates with p and p.adjust annotation (rendered to screen or PNG)
- display data frame (metadat) used for plotting

## How to apply

First run meta.taxa (summary.measure='RR', pool.var='id', studylab='study', backtransform=FALSE, percent.meta=0.5, p.adjust.method='fdr') to obtain metab.sex$random pooled estimates. Then call metatab.show with metatab=metab.sex$random, com.pooled.tab=tabsex4, the target taxonomic level (tax.lev='l2' for phylum, 'l5' family, 'l6' genus), showvar='genderMale', p.cutoff.type='p', p.cutoff=1 and display='data' to build display data rather than a table. Pass this to meta.niceplot with sumtype='taxa', level='main' for phylum or level='sub' for family/genus, p='p', p.adjust='p.adjust', phyla.col='rainbow', and styling options such as forest.col='by.estimate', heat.forest.width.ratio, and alternative palettes (e.g. neg.palette='Greens', pos.palette='Purples', p.sig.heat='yes') to control heatmap/forest proportions and significance emphasis. The rationale is to display pooled log(odds ratio)-based estimates and FDR-adjusted significance jointly so heterogeneity and effect direction across studies (Bangladesh, Haiti, USA(CA_FL), USA(NC)) are visible in one figure.

## Related tools

- **metamicrobiomeR** (provides metatab.show (display='data') and meta.niceplot functions used to build and render the heatmap-forest plots of pooled taxa meta-analysis results) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (runtime environment (R 3.4.2 or higher) in which the plotting workflow is executed)

## Examples

```
meta.niceplot(metadat=metadat,sumtype="taxa",level="main",p="p",p.adjust="p.adjust",phyla.col="rainbow",forest.col="by.estimate",leg.key.size=1,leg.text.size=8,heat.text.x.size=7,heat.text.x.angle=0,forest.axis.text.y=8,forest.axis.text.x=7,point.ratio=c(4,2),line.ratio=c(2,1))
```

## Evaluation signals

- metatab.show is called with display='data' and p.cutoff=1 (not display='table', p.cutoff=0.05) so all taxa enter the figure, not only significant ones
- meta.niceplot is invoked with sumtype='taxa', p='p', p.adjust='p.adjust', and level='main' for phylum or level='sub' for family/genus, matching the taxonomy level of the metadat
- phyla.col='rainbow' yields distinct phylum color keys shared between heatmap and forest panels
- Resulting figure should visually separate pooled estimates by direction (positive/negative) with FDR-adjusted p-values annotated, mirroring the paper's Figures 5/6 style

## Limitations

- Plot options and defaults differ between the GitHub version and the newer CRAN version of metamicrobiomeR, which users are recommended to use
- Rendering requires prior successful execution of meta.taxa and metatab.show; plotting cannot proceed from raw per-study taxa.compare tables alone
- Tutorial details are in the Supplementary file (Additional file 1) rather than fully inline in the manuscript

## Evidence

- [methods] meta.niceplot: "meta.niceplot(metadat=metadat,sumtype="taxa",level="main",p="p",p.adjust="p.adjust",phyla.col="rainbow",forest.col="by.estimate",leg.key.size=1,leg.text.size=8,heat.text.x.size=7"
- [readme] metatab.show display data call: "metadat<-metatab.show(metatab=metab.sex$random,com.pooled.tab=tabsex4,tax.lev="l2",showvar="genderMale",p.cutoff.type="p", p.cutoff=1,display="data")"
- [readme] genus-level plot options: "# Plot with some different options: pooled estimates in forest plot with the same color scales as heatmap, those with p-values<0.05 in bold, FDR adjusted p-values<0.1 in triangles"
- [abstract] package scope: [evidence span withheld — anchor status: not_found]
