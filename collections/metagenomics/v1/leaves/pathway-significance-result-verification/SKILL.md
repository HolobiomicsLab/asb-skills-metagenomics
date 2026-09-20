---
name: pathway-significance-result-verification
description: Use when a PICRUSt2/ggpicrust2 differential abundance run must be validated
  against a published significant-pathway result (e.g., ko05016 and ko05012 under
  LinDA) before the output table is accepted.
when_to_use_negative:
- Input is already a validated feature table or an existing DA results file that only
  needs re-plotting rather than re-testing significance
- The research question calls for pathway-set (enrichment) testing rather than single-pathway
  differential testing — use GSEA (pathway_gsea()) instead
- The 16S data has not yet been functionally predicted; PICRUSt2 prediction and KO-to-pathway
  conversion must precede this verification
tools:
- name: ggpicrust2
  role: R package providing pathway_daa() (LinDA, ALDEx2, etc.), ko2kegg_abundance()
    conversion, and result visualization used to produce the significant-pathway table
    under verification
  repo: https://github.com/cafferychen777/ggpicrust2
- name: PICRUSt2
  role: Predicts metagenome functional content (KO profiles) from the 16S rRNA amplicon
    data upstream of pathway differential testing
- name: limma
  role: Bioconductor dependency underlying linear-model-based differential abundance
    and gene set testing methods in ggpicrust2
- name: devtools
  role: Used to install the ggpicrust2 development version from GitHub
  repo: https://github.com/cafferychen777/ggpicrust2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:f2750d3fafb43551a67dccc046f541409ecc8d149fd70705a5472c4f6092716a
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pathway-significance-result-verification

## Summary

Verifies that differential abundance analysis of PICRUSt2-predicted KEGG pathways (e.g., via ggpicrust2's LinDA) yields the expected significant pathway set — such as ko05016 (Huntington's disease) and ko05012 (Parkinson's disease) in the C9orf72 mouse study — by checking the significant-pathway table for named pathway IDs, descriptions, log fold changes, p-values, and adjusted p-values.

## When to use

Use when a PICRUSt2/ggpicrust2 differential abundance run must be validated against a published significant-pathway result (e.g., ko05016 and ko05012 under LinDA) before the output table is accepted.

## When NOT to use

- Input is already a validated feature table or an existing DA results file that only needs re-plotting rather than re-testing significance
- The research question calls for pathway-set (enrichment) testing rather than single-pathway differential testing — use GSEA (pathway_gsea()) instead
- The 16S data has not yet been functionally predicted; PICRUSt2 prediction and KO-to-pathway conversion must precede this verification

## Inputs

- PICRUSt2-predicted KO abundance table (e.g., the ko_abundance data shipped with ggpicrust2)
- Sample metadata table mapping samples to comparison groups (e.g., prosurvival vs proinflammatory, with/without fecal microbiota transplantation)
- KO-to-KEGG-pathway abundance conversion output (pathway-level feature table)
- Published reference list of expected significant pathways (ko05016, ko05012 per Burberry et al. 2020, Section 2.7 / Fig. 1)

## Outputs

- linda_significant_pathways.csv — significant-pathway table with pathway IDs, descriptions, log fold changes, p-values, and adjusted p-values
- Full differential abundance results table from pathway_daa() for the chosen method (e.g., LinDA)
- Pass/fail validation report confirming ko05016 and ko05012 are among the significant rows

## How to apply

Run the ggpicrust2 pipeline end-to-end: retrieve the deposited 16S rRNA data and metadata, run PICRUSt2, convert KO profiles to KEGG pathway-level abundance with ko2kegg_abundance(), then apply pathway_daa() with daa_method = 'LinDA' (or the default 'camera', which accounts for inter-gene correlations and yields more reliable p-values). Filter the results to significant pathways and write them to a CSV (e.g., linda_significant_pathways.csv) containing pathway IDs, descriptions, log fold changes, p-values, and adjusted p-values. Verification then checks three invariants: the output table exists, ko05016 (Huntington's disease) and ko05012 (Parkinson's disease) appear among its significant rows, and the remaining rows match the significant pathways reported in Section 2.7 / Fig. 1 of the source paper. Optionally run multiple DA methods and compare them with compare_daa_results() to assess robustness of the significant-pathway set.

## Related tools

- **ggpicrust2** (R package providing pathway_daa() (LinDA, ALDEx2, etc.), ko2kegg_abundance() conversion, and result visualization used to produce the significant-pathway table under verification) — https://github.com/cafferychen777/ggpicrust2
- **PICRUSt2** (Predicts metagenome functional content (KO profiles) from the 16S rRNA amplicon data upstream of pathway differential testing)
- **limma** (Bioconductor dependency underlying linear-model-based differential abundance and gene set testing methods in ggpicrust2)
- **devtools** (Used to install the ggpicrust2 development version from GitHub) — https://github.com/cafferychen777/ggpicrust2

## Examples

```
daa_results <- pathway_daa(abundance = ko_abundance, metadata = metadata, group = "environment", daa_method = "LinDA")
```

## Evaluation signals

- linda_significant_pathways.csv exists and is non-empty
- ko05016 (Huntington's disease) and ko05012 (Parkinson's disease) both appear among the significant rows of the output table
- The table contains required columns: pathway ID, description, log fold change, p-value, and adjusted p-value
- The full significant-pathway set matches the pathways reported in Section 2.7 and Fig. 1 of the source paper
- Cross-method comparison (compare_daa_results()) shows the significant findings are not artifacts of a single DA method

## Limitations

- The ggpicrust2 discussion recommends the camera method (default) for most analyses because it provides more reliable p-values by accounting for inter-gene correlations; LinDA results should be interpreted with this in mind
- No software version numbers for ggpicrust2 or PICRUSt2 are stated in the article, so exact reproducibility of significance calls may vary across releases
- Covariate adjustment is recommended when confounding factors like age, sex, or BMI are present in the study design; unadjusted LinDA models may misattribute significance

## Evidence

- [other] Filter the LinDA results for significant pathways and confirm that ko05016 (Huntington's disease) and ko05012 (Parkinson's disease) are significantly different: [evidence span withheld — anchor status: not_found]
- [abstract] This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals.: "This powerful tool enables researchers to conduct extensive differential abundance analyses and generate visually appealing visualizations that effectively highlight functional signals."
- [discussion] camera method provides more reliable p-values by accounting for inter-gene correlations: "camera method provides more reliable p-values by accounting for inter-gene correlations"
- [readme] ko2kegg_abundance(): [evidence span withheld — anchor status: external_doc]
