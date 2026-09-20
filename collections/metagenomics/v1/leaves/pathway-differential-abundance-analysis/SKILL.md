---
name: pathway-differential-abundance-analysis
description: Use when you have a PICRUSt2-derived KEGG pathway abundance table plus
  sample metadata with a grouping variable and need to test which predicted pathways
  differ in abundance between comparison groups.
when_to_use_negative:
- Input is still a KO-level abundance table — convert KO profiles to KEGG pathway-level
  abundance first (ko2kegg_abundance) before running pathway_daa()
- You need ANCOM or ANCOMBC — these methods are explicitly excluded from pathway_daa()'s
  supported set
- The goal is pathway-set-level enrichment rather than single-feature pathway testing
  — use pathway_gsea() instead, which complements DAA for PICRUSt2 functional data
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: ggpicrust2
  role: Provides pathway_daa() to run differential abundance analysis on PICRUSt2
    predicted pathway abundances
  repo: https://github.com/cafferychen777/ggpicrust2
- name: ALDEx2
  role: Optional Bioconductor DA method backend invoked through pathway_daa(daa_method
    = 'ALDEx2')
  repo: https://github.com/cafferychen777/ggpicrust2
- name: LinDA
  role: Log-linear model-based DA method backend invoked through pathway_daa(daa_method
    = 'LinDA')
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Execution environment in which ggpicrust2 and its DA backends run
  repo: https://github.com/cafferychen777/ggpicrust2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:306680ec0058e0679e4ca8a31ecbebc6eaf99f199db4da31003513923cf69437
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - ALDEx2
  - LinDA
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pathway-differential-abundance-analysis

## Summary

Runs differential abundance (DA) analysis on PICRUSt2-predicted KEGG pathway abundance tables using ggpicrust2's pathway_daa(), which integrates nearly all applicable DA methods (ALDEx2, LinDA, etc.) while explicitly excluding ANCOM and ANCOMBC, to test pathway-level differences between comparison groups.

## When to use

Use when you have a PICRUSt2-derived KEGG pathway abundance table plus sample metadata with a grouping variable and need to test which predicted pathways differ in abundance between comparison groups.

## When NOT to use

- Input is still a KO-level abundance table — convert KO profiles to KEGG pathway-level abundance first (ko2kegg_abundance) before running pathway_daa()
- You need ANCOM or ANCOMBC — these methods are explicitly excluded from pathway_daa()'s supported set
- The goal is pathway-set-level enrichment rather than single-feature pathway testing — use pathway_gsea() instead, which complements DAA for PICRUSt2 functional data

## Inputs

- KEGG pathway abundance table (PICRUSt2 predicted functional profile, e.g. from ko2kegg_abundance conversion of KO abundances)
- Sample metadata data frame with a grouping variable defining comparison groups

## Outputs

- Per-method DA result object from pathway_daa() (e.g. ALDEx2, LinDA)
- Tidy per-method CSV artifacts with feature, method, p_val, and p_adjust columns (e.g. daa_results_aldex2.csv, daa_results_linda.csv)

## How to apply

Load the KEGG pathway abundance table (typically produced by chaining ko2kegg_abundance() on PICRUSt2 KO abundances) and the sample metadata in R with ggpicrust2 installed from GitHub (devtools::install_github('cafferychen777/ggpicrust2')). Invoke pathway_daa() once per method, passing the abundance table, metadata, and grouping variable — e.g. first with daa_method = 'ALDEx2' (requires the optional Bioconductor package ALDEx2) and then with daa_method = 'LinDA' to apply log-linear model-based DA testing; the same call pattern applies to the other supported methods. Note that ANCOM and ANCOMBC are intentionally not available through pathway_daa(). Convert each returned result object into a tidy table with columns feature, method, p_val, and p_adjust, and save one CSV per method. Optionally, compare consistency across methods with compare_daa_results() and annotate/visualize significant pathways with pathway_annotation() and pathway_errorbar()/pathway_volcano(). The package recommends covariate adjustment when confounders like age, sex, or BMI are present in the design.

## Related tools

- **ggpicrust2** (Provides pathway_daa() to run differential abundance analysis on PICRUSt2 predicted pathway abundances) — https://github.com/cafferychen777/ggpicrust2
- **ALDEx2** (Optional Bioconductor DA method backend invoked through pathway_daa(daa_method = 'ALDEx2')) — https://github.com/cafferychen777/ggpicrust2
- **LinDA** (Log-linear model-based DA method backend invoked through pathway_daa(daa_method = 'LinDA')) — https://github.com/cafferychen777/ggpicrust2
- **R** (Execution environment in which ggpicrust2 and its DA backends run) — https://github.com/cafferychen777/ggpicrust2

## Examples

```
daa_results <- pathway_daa(abundance = kegg_abundance, metadata = metadata, group = "group", daa_method = "ALDEx2")
```

## Evaluation signals

- Each output CSV exists, is parseable, and contains exactly the columns feature, method, p_val, and p_adjust with non-empty rows
- One CSV artifact is produced per DA method (daa_results_aldex2.csv, daa_results_linda.csv), each traceable to a single pathway_daa() invocation
- p_adjust values fall in [0,1] and p_val <= p_adjust per row (Bonferroni/FDR-style correction invariant)
- No rows reference ANCOM or ANCOMBC in the method column, since pathway_daa() excludes them
- The feature column contains KEGG pathway identifiers matching the input abundance table's rows

## Limitations

- pathway_daa() does not support ANCOM or ANCOMBC, so those methods cannot be used within this workflow
- ALDEx2 (and other Bioconductor-backed methods) require optional Bioconductor package installation beyond ggpicrust2's minimal CRAN dependency set
- The article notes no state-of-the-art inference tools were previously available to test differences in PICRUSt2 pathway abundances, so cross-tool comparisons may be limited; ggpicrust2's method-agreement comparison is intended to evaluate robustness of findings
- No software version numbers are stated for ggpicrust2 in the article, so exact behavior may vary across package versions

## Evidence

- [intro] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results.: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [abstract] However, no state-of-art inference tools are available to test the differences in pathway abundances between comparison groups.: "However, no state-of-art inference tools are available to test the differences in pathway abundances between comparison groups."
- [readme] ggpicrust2() integrates ko abundance to kegg pathway abundance conversion, annotation of pathway, differential abundance (DA) analysis, part of DA results visualization.: [evidence span withheld — anchor status: external_doc]
- [readme] This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome of a particular sample.: "This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome of a particular sample."
