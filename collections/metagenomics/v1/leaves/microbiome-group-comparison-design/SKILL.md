---
name: microbiome-group-comparison-design
description: Use when you have a PICRUSt2-predicted pathway-level abundance table
  (KO converted to KEGG pathway abundance) plus sample metadata with a grouping variable,
  and you must test which pathways differ significantly between groups.
when_to_use_negative:
- Input is raw 16S rRNA reads not yet processed by PICRUSt2 — functional prediction
  must come first.
- The goal is pathway-set-level enrichment rather than single-feature differential
  abundance — use the GSEA workflow (pathway_gsea()) instead.
- The goal is to compare agreement between DA methods on the same dataset rather than
  test group differences — use compare_daa_results()/compare_metagenome_results()
  instead.
tools:
- name: ggpicrust2
  role: Provides pathway_daa() to run differential abundance analysis on PICRUSt2
    predicted pathway abundance tables and compare_daa_results() for cross-method
    consistency checks.
  repo: https://github.com/cafferychen777/ggpicrust2
- name: ALDEx2
  role: Advanced DA backend supported by pathway_daa() for testing pathway-level differential
    abundance between comparison groups.
- name: LinDA
  role: Log-linear model-based DA backend supported by pathway_daa(); used to identify
    significant KEGG pathways (ko05016, ko05012) in the C9orf72 application.
- name: PICRUSt2
  role: Upstream tool that predicts metagenome functional content from 16S rRNA gene
    data, producing the abundance profiles analyzed here.
- name: limma
  role: Bioconductor dependency powering camera/fry competitive gene set testing with
    inter-gene correlation adjustment and covariate adjustment.
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:a6b53e0cc5ee235166af08914ea61c1e87761288f22b16c50ec13c183efae5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - ALDEx2
  - LinDA
  - PICRUSt2
  - R
  - devtools
  - ggpicrust2
  - limma
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# microbiome-group-comparison-design

## Summary

Designs group-comparison differential abundance analysis of PICRUSt2-predicted pathway profiles (e.g., KEGG pathways from 16S rRNA-derived metagenomes) using ggpicrust2's pathway_daa(), which integrates advanced DA methods such as LinDA and ALDEx2 so that significant pathways between comparison groups can be identified and compared for robustness.

## When to use

Use when you have a PICRUSt2-predicted pathway-level abundance table (KO converted to KEGG pathway abundance) plus sample metadata with a grouping variable, and you must test which pathways differ significantly between groups.

## When NOT to use

- Input is raw 16S rRNA reads not yet processed by PICRUSt2 — functional prediction must come first.
- The goal is pathway-set-level enrichment rather than single-feature differential abundance — use the GSEA workflow (pathway_gsea()) instead.
- The goal is to compare agreement between DA methods on the same dataset rather than test group differences — use compare_daa_results()/compare_metagenome_results() instead.

## Inputs

- PICRUSt2-predicted KO abundance table (e.g., bundled ko_abundance)
- Sample metadata with a grouping variable (e.g., bundled metadata)
- KEGG pathway-level abundance table obtained from KO-to-pathway conversion

## Outputs

- Per-method DA result tables with feature, method, p_val, and p_adjust columns (e.g., daa_results_linda.csv, daa_results_aldex2.csv)
- Filtered significant-pathway table with pathway IDs, descriptions, log fold changes, p-values, and adjusted p-values (e.g., linda_significant_pathways.csv)

## How to apply

Start from the KO-level abundance table and convert KO profiles to KEGG pathway-level abundance (ko2kegg_abundance) so pathway_daa() can operate on pathway features. Call pathway_daa() with the pathway abundance table, sample metadata, the grouping variable, and a chosen daa_method (e.g., LinDA or ALDEx2); note that ANCOM and ANCOMBC are explicitly excluded by the package. Convert the returned result object into a tidy table with feature, method, p_val, and p_adjust columns, and filter for significant pathways (e.g., ko05016 and ko05012 in the C9orf72 case study). Use compare_daa_results()/compare_metagenome_results() to check agreement and discrepancy across methods applied to the same dataset, as a robustness check on findings. Prefer the camera method for gene-set/GSEA-style testing because it accounts for inter-gene correlations, and adjust for covariates such as age, sex, or BMI when confounders are present in the study design.

## Related tools

- **ggpicrust2** (Provides pathway_daa() to run differential abundance analysis on PICRUSt2 predicted pathway abundance tables and compare_daa_results() for cross-method consistency checks.) — https://github.com/cafferychen777/ggpicrust2
- **ALDEx2** (Advanced DA backend supported by pathway_daa() for testing pathway-level differential abundance between comparison groups.)
- **LinDA** (Log-linear model-based DA backend supported by pathway_daa(); used to identify significant KEGG pathways (ko05016, ko05012) in the C9orf72 application.)
- **PICRUSt2** (Upstream tool that predicts metagenome functional content from 16S rRNA gene data, producing the abundance profiles analyzed here.)
- **limma** (Bioconductor dependency powering camera/fry competitive gene set testing with inter-gene correlation adjustment and covariate adjustment.)

## Examples

```
data(ko_abundance); data(metadata); daa_results <- pathway_daa(abundance = ko_abundance, metadata = metadata, group = "group", daa_method = "LinDA")
```

## Evaluation signals

- Each output CSV exists, is parseable, and contains the feature, method, p_val, and p_adjust columns with non-empty rows.
- The significant-pathway table contains the expected significant features (e.g., ko05016 and ko05012 must appear among significant rows in the C9orf72 case study) and otherwise matches pathways reported in the source paper's results/figures.
- Log fold changes, p-values, and adjusted p-values are present and numerically plausible for every significant row.
- Cross-method comparison (e.g., LinDA vs ALDEx2) shows interpretable agreement/discrepancy, which the package exposes as a robustness signal rather than an error.
- DA methods used are among those pathway_daa() supports (LinDA, ALDEx2, etc.); ANCOM and ANCOMBC should be absent from the method column.

## Limitations

- ANCOM and ANCOMBC are explicitly excluded from the DA methods supported by pathway_daa().
- No software version numbers are stated for ggpicrust2 or PICRUSt2 in the article, complicating exact reproduction.
- Preranked GSEA-style tests can produce unreliable p-values because they do not account for inter-gene correlations; camera is recommended as the default instead.
- Covariate adjustment is recommended when confounding factors like age, sex, or BMI are present; ignoring them may bias results.

## Evidence

- [abstract] ggpicrust2 enables extensive differential abundance analyses and visualizations of functional profiles derived from 16S rRNA sequencing: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations"
- [intro] ggpicrust2 can compare consistency across different DA methods applied to the same dataset: "This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome"
- [discussion] camera provides more reliable p-values by accounting for inter-gene correlations: "it provides more reliable p-values by accounting for inter-gene correlations"
- [discussion] covariate adjustment recommended for confounders: "**Adjust for covariates** when you have confounding factors like age, sex, or BMI in your study design"
- [readme] ggpicrust2 main pipeline integrates KO-to-pathway conversion and DA analysis: [evidence span withheld — anchor status: external_doc]
