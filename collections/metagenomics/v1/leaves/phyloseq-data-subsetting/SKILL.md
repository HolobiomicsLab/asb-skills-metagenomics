---
name: phyloseq-data-subsetting
description: Use when analysis must be restricted to a treatment-arm subgroup of a
  phyloseq microbiome dataset (e.g., control-group survival association in MiSurv
  Module 3) and metadata columns define the subgroup.
when_to_use_negative:
- Analysis should span all treatment groups (no subgroup option selected in MiSurv)
  — subsetting would discard samples needed for the full-cohort comparison
- Input is raw sequence data or a QIIME2 feature table not yet converted to a phyloseq
  object; the data processing module must run first
- The grouping variable is absent or ambiguous in sam.dat.txt, making subgroup membership
  undefined
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: R package holding the biom.Rdata object and providing the subsetting and distance
    functions applied to the subsetted OTU table
  repo: https://github.com/wg99526/MiSurvGit
- name: MiSurv
  role: Web cloud platform / shiny app whose subgroup analysis option consumes the
    subsetted data for Modules 2–4
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:73fdba34be10b8c81894aa90302bdc00d45ac9a913e59421c2a8f5e9024f5ae2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - phyloseq
  - shiny
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# phyloseq-data-subsetting

## Summary

Subsetting a phyloseq OTU/sample object to a subgroup (e.g., control-group subjects) of the antibiotic-T1D mouse trial data before downstream MiSurv analyses, using the sample metadata columns (group, survival time, event, Sex).

## When to use

Use when analysis must be restricted to a treatment-arm subgroup of a phyloseq microbiome dataset (e.g., control-group survival association in MiSurv Module 3) and metadata columns define the subgroup.

## When NOT to use

- Analysis should span all treatment groups (no subgroup option selected in MiSurv) — subsetting would discard samples needed for the full-cohort comparison
- Input is raw sequence data or a QIIME2 feature table not yet converted to a phyloseq object; the data processing module must run first
- The grouping variable is absent or ambiguous in sam.dat.txt, making subgroup membership undefined

## Inputs

- biom.Rdata (phyloseq object from MiSurvGit Data folder)
- otu.tab.txt (OTU count table)
- sam.dat.txt (sample metadata with group, survival time, event, Sex)
- tax.tab.txt (taxonomic annotation table)
- tree.tre (phylogenetic tree)

## Outputs

- Subsetted phyloseq object restricted to the target subgroup (e.g., control-group subjects)
- Subsetted OTU table and metadata ready for MiSurv Module 3 association testing or Module 2 composition analysis

## How to apply

Load the processed study data from the MiSurv GitHub Data folder, either the 'phyloseq' object 'biom.Rdata' or the four individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre'. Use the group column in the sample metadata ('sam.dat.txt') to subset the phyloseq object to only the subjects of interest (e.g., control group), while retaining the survival time, censored/event status, and covariate columns (e.g., Sex). The subgroup option exists because the effects of the microbiome on survival responses may differ between treatment groups, so subgroup-level analyses (MiSurv Modules 3 and 4) require a correctly filtered phyloseq object before computing diversity metrics such as Jaccard dissimilarity. Verify the subset preserves all sample-level columns needed by the downstream MiSurv module so covariate adjustment (Sex) and survival responses carry through.

## Related tools

- **phyloseq** (R package holding the biom.Rdata object and providing the subsetting and distance functions applied to the subsetted OTU table) — https://github.com/wg99526/MiSurvGit
- **MiSurv** (Web cloud platform / shiny app whose subgroup analysis option consumes the subsetted data for Modules 2–4) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(phyloseq); ps <- readRDS('biom.Rdata'); ps_ctrl <- subset_samples(ps, group == 'control')
```

## Evaluation signals

- The subsetted sample metadata retains survival time, censored/event status, and Sex columns for every retained subject
- Subset sample count equals the number of control-group subjects in sam.dat.txt (group column filter applied without dropping or duplicating samples)
- Downstream MiRKAT-S on the subset Jaccard kernel reproduces the published control-group P value of 0.046 within ±0.01
- OTU table rows/features are unchanged by subsetting; only samples are filtered

## Limitations

- MiSurv local implementation requires R (>4.2.0), so older R environments may fail when loading/manipulating the phyloseq object
- Subgroup analysis assumes microbiome effects on survival differ between treatment groups; if the subgroup is too small, survival association tests may be underpowered
- No version numbers are given for the R packages used (e.g., phyloseq), so reproducibility of subsetting behavior across package versions is not guaranteed

## Evidence

- [other] Load the processed study data from the MiSurv repository Data folder, either the 'phyloseq' object 'biom.Rdata' or the equivalent individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre'.: [evidence span withheld — anchor status: not_found]
- [other] Subset the phyloseq object to only the control-group subjects using the group column of the sample metadata in 'sam.dat.txt', keeping the survival time, event status, and Sex covariate columns.: [evidence span withheld — anchor status: not_found]
- [discussion] we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] survival data and analytic plans (survival time, censored/event, follow-up period, subgroup analysis): "survival data and analytic plans (survival time, censored/event, follow-up period, subgroup analysis)"
