---
name: multiple-testing-pvalue-adjustment
description: Use when DAA results on pathway-level KEGG abundance tables contain raw
  p_val columns and significance must be judged on p_adjust, because hundreds of pathways
  are tested at once.
when_to_use_negative:
- Input is a raw KO abundance table that has not yet been converted to KEGG pathway-level
  abundance — ko2kegg_abundance() must run first
- The analysis goal is gene-set-level enrichment (use pathway_gsea() instead of per-pathway
  DAA testing)
- Fewer than a handful of features are tested, so multiple-testing correction would
  be unnecessary or uninformative
edam_operation: http://edamontology.org/operation_3812
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ggpicrust2
  role: Provides pathway_daa() which runs DAA on PICRUSt2 predicted pathway abundances
    and returns p_val/p_adjust per pathway, plus compare_daa_results() for cross-method
    consistency checks
  repo: https://github.com/cafferychen777/ggpicrust2
- name: ALDEx2
  role: Optional Bioconductor-backed DA method invoked through pathway_daa() whose
    per-pathway p-values are corrected to p_adjust
- name: LinDA
  role: Log-linear model-based DA method invoked through pathway_daa() as a second
    correction/robustness comparison against ALDEx2
- name: PICRUSt2
  role: Produces the predicted functional (KO/pathway) profiles that the adjusted
    DAA tests operate on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:f88087b7bcaa730512f3a8e468abf036263d16b3d3276c214572c8da1e3ac437
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - ALDEx2
  - LinDA
  - PICRUSt2
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_3812
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# multiple-testing-pvalue-adjustment

## Summary

Adjusts per-pathway p-values from differential abundance analysis (DAA) of PICRUSt2-predicted functional profiles for multiple hypothesis testing, so that the p_adjust column of ggpicrust2's pathway_daa() results reflects corrected significance across the many KEGG pathway features tested simultaneously.

## When to use

Use when DAA results on pathway-level KEGG abundance tables contain raw p_val columns and significance must be judged on p_adjust, because hundreds of pathways are tested at once.

## When NOT to use

- Input is a raw KO abundance table that has not yet been converted to KEGG pathway-level abundance — ko2kegg_abundance() must run first
- The analysis goal is gene-set-level enrichment (use pathway_gsea() instead of per-pathway DAA testing)
- Fewer than a handful of features are tested, so multiple-testing correction would be unnecessary or uninformative

## Inputs

- KEGG pathway abundance table (PICRUSt2-predicted, converted from ko_abundance)
- Sample metadata with a grouping variable (e.g. the bundled R metadata dataset)
- DA method choice (camera default; ALDEx2, LinDA supported; ANCOM/ANCOMBC excluded)

## Outputs

- Per-method DAA result table with feature, method, p_val, p_adjust columns
- CSV artifacts such as daa_results_aldex2.csv and daa_results_linda.csv
- Cross-method comparison of adjusted results via compare_daa_results()

## How to apply

Run ggpicrust2's pathway_daa() on the KEGG pathway abundance table (produced from ko_abundance via ko2kegg_abundance()) with a chosen DA method — the card workflow uses ALDEx2 and LinDA, while camera is the README's recommended default and ANCOM/ANCOMBC are excluded. Each run returns a result table containing raw p_val and a multiple-testing-corrected p_adjust column; the adjustment is computed across all pathway features within the single method's test set. Convert each method's result to a tidy table with columns feature, method, p_val, and p_adjust and export per-method CSVs (e.g. daa_results_aldex2.csv, daa_results_linda.csv). Interpret significance from p_adjust rather than p_val, and when comparing ALDEx2 vs LinDA, compare correction outcomes across methods using compare_daa_results()/compare_metagenome_results() to assess robustness. Adjust covariates (age, sex, BMI) in the DA design when confounding factors are present, per the paper's discussion.

## Related tools

- **ggpicrust2** (Provides pathway_daa() which runs DAA on PICRUSt2 predicted pathway abundances and returns p_val/p_adjust per pathway, plus compare_daa_results() for cross-method consistency checks) — https://github.com/cafferychen777/ggpicrust2
- **ALDEx2** (Optional Bioconductor-backed DA method invoked through pathway_daa() whose per-pathway p-values are corrected to p_adjust)
- **LinDA** (Log-linear model-based DA method invoked through pathway_daa() as a second correction/robustness comparison against ALDEx2)
- **PICRUSt2** (Produces the predicted functional (KO/pathway) profiles that the adjusted DAA tests operate on)

## Examples

```
res_aldex2 <- pathway_daa(abundance = kegg_abundance, metadata = metadata, group = "group", daa_method = "ALDEx2"); res_linda <- pathway_daa(abundance = kegg_abundance, metadata = metadata, group = "group", daa_method = "LinDA")
```

## Evaluation signals

- Each output CSV (daa_results_aldex2.csv, daa_results_linda.csv) exists, parses, and contains the feature, method, p_val, and p_adjust columns with non-empty rows
- p_adjust values are non-decreasing relative to p_val within each method's result table (0 <= p_val <= p_adjust <= 1)
- For every method tested, each feature row carries the correct method label and both p_val and p_adjust are numeric and non-missing
- Adjusted significance calls (small p_adjust) agree reasonably with the unadjusted ranking; discrepancies across ALDEx2 and LinDA are inspected via the method-comparison feature rather than silently ignored

## Limitations

- ANCOM and ANCOMBC are explicitly excluded from ggpicrust2's supported DA methods, so adjustment is only available for the methods pathway_daa() integrates
- Camera is recommended as the default method for most analyses because it gives more reliable p-values by accounting for inter-gene correlations; other methods may yield less reliable p-values
- Preranked GSEA-style approaches can produce 'spectacularly wrong p-values' when inter-gene correlations are ignored — an analogous caution for interpreting uncorrected per-feature statistics
- No software version numbers are stated for ggpicrust2 or PICRUSt2, and no seed/randomness control is documented, complicating exact reproduction of p_adjust values

## Evidence

- [intro] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results.: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
- [readme] This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome.: "This feature allows users to assess the agreement and discrepancy between various methods when it comes to predicting and sequencing the metagenome"
- [readme] ALDEx2 | Differential abundance analysis of taxonomic and functional features: [evidence span withheld — anchor status: external_doc]
- [abstract] no state-of-art inference tools are available to test the differences in pathway abundances between comparison groups: "no state-of-art inference tools are available to test the differences in pathway abundances between comparison groups."
