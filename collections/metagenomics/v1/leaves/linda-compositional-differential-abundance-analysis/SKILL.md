---
name: linda-compositional-differential-abundance-analysis
description: Use when a two-group comparison of PICRUSt2-predicted KO/KEGG pathway
  abundances (e.g., prosurvival vs proinflammatory fecal-transplant groups) must be
  tested for significant differential pathways with a compositional DA method like
  LinDA.
when_to_use_negative:
- Input is raw 16S rRNA reads or OTU/ASV tables — PICRUSt2 functional prediction and
  KO-to-pathway conversion must precede LinDA testing.
- The goal is taxa-level (not pathway-level) differential abundance of the sequencing
  data itself rather than predicted functional profiles.
- Only single-group or paired longitudinal designs are present — LinDA here is applied
  to between-group comparisons defined in the metadata.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_0091
tools:
- name: ggpicrust2
  role: R package that implements LinDA and other differential abundance methods on
    PICRUSt2-predicted functional profiles and visualizes the results
  repo: https://github.com/cafferychen777/ggpicrust2
- name: PICRUSt2
  role: Predicts metagenome functional content (KO abundances) from 16S rRNA gene
    data prior to LinDA pathway testing
- name: limma
  role: Bioconductor dependency underpinning linear-model-based testing in ggpicrust2
    (e.g., camera/fry methods used for consistency checks)
- name: devtools
  role: Installs the ggpicrust2 development version from GitHub
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment in which ggpicrust2, LinDA, and the downstream filtering/export
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:ab969dcad0906efcbe50879989a69235288b3618d4c573603d0aeeb5d6ac4e9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - PICRUSt2
  - R
  - devtools
  - ggpicrust2
  - limma
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# linda-compositional-differential-abundance-analysis

## Summary

Differential abundance analysis of PICRUSt2-predicted KEGG pathway profiles using the LinDA method implemented in the ggpicrust2 R package, as demonstrated on C9orf72 mouse 16S rRNA data where LinDA identified ko05016 and ko05012 as significantly different between transplantation groups.

## When to use

Use when a two-group comparison of PICRUSt2-predicted KO/KEGG pathway abundances (e.g., prosurvival vs proinflammatory fecal-transplant groups) must be tested for significant differential pathways with a compositional DA method like LinDA.

## When NOT to use

- Input is raw 16S rRNA reads or OTU/ASV tables — PICRUSt2 functional prediction and KO-to-pathway conversion must precede LinDA testing.
- The goal is taxa-level (not pathway-level) differential abundance of the sequencing data itself rather than predicted functional profiles.
- Only single-group or paired longitudinal designs are present — LinDA here is applied to between-group comparisons defined in the metadata.

## Inputs

- PICRUSt2-predicted KO abundance table (e.g., ko_abundance-style feature table)
- Sample metadata with group labels (e.g., prosurvival vs proinflammatory transplantation groups)
- KEGG pathway-level abundance table converted from KO profiles
- Pathway name/description annotation for KEGG pathway IDs

## Outputs

- LinDA differential abundance results per pathway (pathway ID, description, log fold change, p-value, adjusted p-value)
- Filtered significant-pathway table (e.g., linda_significant_pathways.csv)
- Visualizations of DA results (e.g., pathway_errorbar, heatmap, volcano plot)

## How to apply

Start from PICRUSt2 output derived from 16S rRNA gene data, and convert KO abundance profiles to KEGG pathway-level abundance with ko2kegg_abundance() when needed, as ggpicrust2 supports. In R, install ggpicrust2 from GitHub via devtools, then run pathway_daa() with daa_method = 'LinDA', passing the pathway abundance table, sample metadata, and the grouping column (e.g., transplantation group). Filter the returned LinDA results for significant pathways, keeping pathway IDs, descriptions, log fold changes, p-values, and adjusted p-values, and write them to a table such as linda_significant_pathways.csv. For the C9orf72 use case, confirm ko05016 (Huntington's disease) and ko05012 (Parkinson's disease) appear among the significant rows, matching Section 2.7 / Fig. 1 of the source paper. Because ggpicrust2 can compare consistency across DA methods, cross-check LinDA results against other methods (e.g., ALDEx2, edgeR, limma-based camera/fry) to evaluate robustness of findings.

## Related tools

- **ggpicrust2** (R package that implements LinDA and other differential abundance methods on PICRUSt2-predicted functional profiles and visualizes the results) — https://github.com/cafferychen777/ggpicrust2
- **PICRUSt2** (Predicts metagenome functional content (KO abundances) from 16S rRNA gene data prior to LinDA pathway testing)
- **limma** (Bioconductor dependency underpinning linear-model-based testing in ggpicrust2 (e.g., camera/fry methods used for consistency checks))
- **devtools** (Installs the ggpicrust2 development version from GitHub) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment in which ggpicrust2, LinDA, and the downstream filtering/export are executed)

## Examples

```
devtools::install_github("cafferychen777/ggpicrust2")
```

## Evaluation signals

- linda_significant_pathways.csv exists and contains ko05016 and ko05012 among its significant rows, matching Section 2.7 and Fig. 1 of the C9orf72 paper.
- The results table includes pathway IDs, descriptions, log fold changes, p-values, and adjusted p-values for each significant pathway.
- The group design passed to LinDA matches the metadata (prosurival vs proinflammatory transplantation groups of C9orf72 loss-of-function mice with and without FMT).
- KO abundances were converted to KEGG pathway-level abundance before testing, so results are keyed by KEGG pathway IDs (ko*) not KO IDs.
- Significant-pathway overlap with alternative DA methods in ggpicrust2 is checked to assess agreement and robustness of findings.

## Limitations

- ggpicrust2's discussion recommends the camera method (default) for most analyses and covariate adjustment for confounders like age, sex, or BMI; LinDA results should be validated against these recommendations.
- No software version numbers are stated for ggpicrust2 or PICRUSt2 in the article, and no seed/randomness control information is documented, complicating exact reproducibility.
- Comparing DA methods may yield agreement and discrepancy across methods applied to the same dataset, so single-method significance (LinDA alone) should not be treated as definitive.

## Evidence

- [readme] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results.: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [abstract] We have developed ggpicrust2, an R package, for analyzing functional profiles derived from 16S rRNA sequencing.: "We have developed ggpicrust2, an R package, for analyzing functional profiles derived from 16S rRNA sequencing."
- [intro] convert KO profiles to pathway-level abundance when needed: "convert KO profiles to pathway-level abundance when needed"
- [readme] This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome of a particular sample.: "This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome of a particular sample."
