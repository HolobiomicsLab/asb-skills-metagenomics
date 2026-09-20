---
name: gene-set-enrichment-analysis
description: Use when analyzing PICRUSt2 KO/predicted functional abundance tables
  and pathway-set (KEGG pathway) level enrichment signals are more appropriate than
  single-feature differential abundance testing.
when_to_use_negative:
- The goal is single-feature (individual pathway/KO) differential abundance testing
  rather than pathway-set enrichment — use pathway_daa() instead
- Gene sets of interest fall outside the 5-500 member size range after filtering
- Preranked statistics are the only available input and inter-gene correlation structure
  cannot be modeled (preranked GSEA can produce unreliable p-values)
edam_operation: http://edamontology.org/operation_3932
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0605
tools:
- name: ggpicrust2
  role: Provides pathway_gsea(), visualize_gsea(), compare_gsea_daa() and gsea_pathway_annotation()
    for running and interpreting GSEA on PICRUSt2 predicted functional data
  repo: https://github.com/cafferychen777/ggpicrust2
- name: limma
  role: Back-end for the camera (default) and fry competitive gene set testing methods
    used by pathway_gsea()
- name: PICRUSt2
  role: Produces the predicted functional (KO) profiles that serve as GSEA input
- name: R
  role: Runtime environment; ggpicrust2 is installed via devtools::install_github
    or CRAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:c031edd1c07b93491438d98daed121fae0e80c243581082ee921db27743b6e08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - PICRUSt2
  - R
  - ggpicrust2
  - limma
  edam_operation: http://edamontology.org/operation_3932
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-set-enrichment-analysis

## Summary

Perform Gene Set Enrichment Analysis (GSEA) on PICRUSt2-predicted functional profiles using ggpicrust2's pathway_gsea(), preferring the limma-based camera method that accounts for inter-gene correlations to obtain more reliable p-values than preranked approaches.

## When to use

Use when analyzing PICRUSt2 KO/predicted functional abundance tables and pathway-set (KEGG pathway) level enrichment signals are more appropriate than single-feature differential abundance testing.

## When NOT to use

- The goal is single-feature (individual pathway/KO) differential abundance testing rather than pathway-set enrichment — use pathway_daa() instead
- Gene sets of interest fall outside the 5-500 member size range after filtering
- Preranked statistics are the only available input and inter-gene correlation structure cannot be modeled (preranked GSEA can produce unreliable p-values)

## Inputs

- PICRUSt2-predicted KO abundance table (ko_abundance)
- sample metadata table (metadata)
- KEGG pathway gene sets built from KO features (sets of size 5-500)

## Outputs

- GSEA results table with pathway ID, NES, p-value and p.adjust columns (e.g. gsea_results.csv)
- annotated GSEA results with pathway descriptions via gsea_pathway_annotation()
- GSEA visualizations (enrichment plots, dot plots, network plots, heatmaps) via visualize_gsea()

## How to apply

Load PICRUSt2-predicted KO abundance and sample metadata (e.g. the bundled data(ko_abundance) and data(metadata)). Build KEGG pathway gene sets from the KO features, filtering gene sets to those with min_size=5 and max_size=500. Run ggpicrust2's pathway_gsea() with method='camera' (the default since v2.5.6), which uses limma's competitive gene set test and accounts for inter-gene correlations; use method='fry' instead when faster computation is needed for large gene set collections. Adjust for covariates (age, sex, BMI) via the design when confounders are present, and specify contrasts for complex experimental designs. Extract the results table with pathway ID, NES, p-value and p.adjust columns for downstream annotation (gsea_pathway_annotation) or comparison with differential abundance results (compare_gsea_daa).

## Related tools

- **ggpicrust2** (Provides pathway_gsea(), visualize_gsea(), compare_gsea_daa() and gsea_pathway_annotation() for running and interpreting GSEA on PICRUSt2 predicted functional data) — https://github.com/cafferychen777/ggpicrust2
- **limma** (Back-end for the camera (default) and fry competitive gene set testing methods used by pathway_gsea())
- **PICRUSt2** (Produces the predicted functional (KO) profiles that serve as GSEA input)
- **R** (Runtime environment; ggpicrust2 is installed via devtools::install_github or CRAN)

## Examples

```
data(ko_abundance); data(metadata); gsea_results <- pathway_gsea(ko_abundance, metadata, method = "camera")
```

## Evaluation signals

- gsea_results.csv exists, parses as a valid CSV, and contains pathway ID, NES, p-value and p.adjust columns
- All retained gene sets have sizes within the min_size=5 to max_size=500 range
- The camera method (default) was used, and p-values are consistent with a test that accounts for inter-gene correlations
- Adjusted p-values (p.adjust) are in [0,1] and ordered consistently with raw p-values
- GSEA results can be cross-checked against differential abundance results via compare_gsea_daa() for agreement/discrepancy assessment

## Limitations

- Preranked GSEA methods can produce 'spectacularly wrong p-values' because they do not account for inter-gene correlations
- Covariates such as age, sex, or BMI should be adjusted for; ignoring confounding factors can bias results
- No software version numbers for ggpicrust2 or PICRUSt2 are stated in the article; camera/fry default behavior applies from package version v2.5.6 per the README

## Evidence

- [discussion] camera method provides more reliable p-values by accounting for inter-gene correlations: "camera method provides more reliable p-values by accounting for inter-gene correlations"
- [discussion] Use camera method (default) for most analyses: "**Use `camera` method (default)** for most analyses"
- [readme] Camera accounts for inter-gene correlations, providing more reliable p-values than preranked methods: "**`camera` method (new default)**: Accounts for inter-gene
  correlations, providing more reliable p-values than preranked methods"
- [intro] The vignette demonstrates GSEA on PICRUSt2 predicted functional data to identify pathways enriched in different conditions: [evidence span withheld — anchor status: not_found]
- [intro] Bundled example datasets ko_abundance and metadata are loaded with data(): "data(ko_abundance)
data(metadata)"
