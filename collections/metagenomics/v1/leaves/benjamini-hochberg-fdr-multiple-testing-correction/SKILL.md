---
name: benjamini-hochberg-fdr-multiple-testing-correction
description: Use when hundreds/thousands of taxa (phylum through genus) each yield
  a raw P-value from group-comparison regression and a family-wise-error-controlled,
  FDR-adjusted significance list is needed.
when_to_use_negative:
- The comparison involves a single hypothesis or a small prespecified set of taxa,
  where a family-wise-error method or no correction may be more appropriate than FDR
  over the full taxon family
- The abundance data have not yet been CLR (centered log-ratio) transformed or the
  regression has not been fit per taxon — this skill corrects P-values only, it does
  not perform the differential abundance testing itself
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: MiSurv
  role: Web cloud platform whose comparative microbial composition analysis module
    (Module 2) fits covariate-adjusted CLR-based taxon regressions and applies Benjamini-Hochberg
    FDR correction at 5%
  repo: https://github.com/wg99526/MiSurvGit
- name: R
  role: Runtime (>4.2.0) in which the per-taxon regressions and p.adjust BH correction
    are executed for local reproduction
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used in the data processing module to rarefy (rarefy_even_depth)
    and CLR-transform the OTU count data feeding the per-taxon tests
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:83f81d5eddc31d3a7cffd0ac3ac58064242ded1abe66fb3bd4b2b76e911157f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - phyloseq
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# benjamini-hochberg-fdr-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg FDR correction at a 5% threshold to raw P-values from per-taxon covariate-adjusted CLR-based differential abundance testing, as used in the MiSurv Module 2 analysis of the control-vs-tylosin example trial, yielding 66 significant taxa from phylum to genus.

## When to use

Use when hundreds/thousands of taxa (phylum through genus) each yield a raw P-value from group-comparison regression and a family-wise-error-controlled, FDR-adjusted significance list is needed.

## When NOT to use

- The comparison involves a single hypothesis or a small prespecified set of taxa, where a family-wise-error method or no correction may be more appropriate than FDR over the full taxon family
- The abundance data have not yet been CLR (centered log-ratio) transformed or the regression has not been fit per taxon — this skill corrects P-values only, it does not perform the differential abundance testing itself

## Inputs

- Vector of raw P-values for the treatment-group coefficient from linear regressions of CLR-transformed abundances (one per taxon, phylum to genus)
- Processed OTU count table (otu.tab.txt) and sample metadata (sam.dat.txt) with Sex and treatment group covariates

## Outputs

- significant_taxa_fdr5.tsv: table of significant taxa (BH-adjusted P < 0.05) with adjusted P-values
- Count of significant taxa (expected: 66) matching Figures 4 and 5

## How to apply

For each taxon from phylum to genus level, fit a linear regression of CLR-transformed abundance on treatment group (control vs tylosin) adjusted for Sex, as implemented in the MiSurv comparative microbial composition analysis module. Collect the raw P-values for the group coefficient across all taxa into a single vector, preserving taxon labels. Apply the Benjamini-Hochberg step-up procedure (e.g., p.adjust(method='BH') in R >4.2.0) at a 5% threshold to control the false discovery rate across the full taxon family. Retain taxa with adjusted P < 0.05, write them with their adjusted P-values to significant_taxa_fdr5.tsv, and confirm the output count matches the paper's reported result (66 significant taxa, consistent with Figures 4 and 5).

## Related tools

- **MiSurv** (Web cloud platform whose comparative microbial composition analysis module (Module 2) fits covariate-adjusted CLR-based taxon regressions and applies Benjamini-Hochberg FDR correction at 5%) — https://github.com/wg99526/MiSurvGit
- **R** (Runtime (>4.2.0) in which the per-taxon regressions and p.adjust BH correction are executed for local reproduction) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used in the data processing module to rarefy (rarefy_even_depth) and CLR-transform the OTU count data feeding the per-taxon tests) — https://github.com/wg99526/MiSurvGit

## Examples

```
adj.p <- p.adjust(raw.p, method = "BH"); sig <- names(adj.p[adj.p < 0.05])  # expect 66 taxa, write to significant_taxa_fdr5.tsv
```

## Evaluation signals

- The output table significant_taxa_fdr5.tsv contains exactly 66 significant taxa rows at BH FDR 5%, matching the paper's Figures 4 and 5
- Every reported adjusted P-value is < 0.05 and monotonically consistent with the rank-ordered raw P-values under the BH step-up procedure
- The set of raw P-values supplied to the correction covers one entry per taxon from phylum to genus (no taxa dropped before correction, which would understate multiplicity)
- The same 66-taxon result is reproduced when Sex is included as a covariate in each taxon's linear regression (covariate-adjusted model)

## Limitations

- The validated result (66 taxa) is specific to the control-vs-tylosin comparison of the QIITA study 10508 data with Sex adjustment; other datasets or covariates will yield different counts
- BH correction operates on raw P-values only and provides no protection if the underlying per-taxon linear models or the CLR transformation are mis-specified
- No software version numbers are given for R packages used in the pipeline, so exact reproduction may require re-validating against Figures 4 and 5

## Evidence

- [methods] Collect the raw P-values for the group coefficient across all taxa and apply Benjamini-Hochberg FDR correction at a 5% threshold.: [evidence span withheld — anchor status: not_found]
- [results] 66 taxa from phylum to genus showed significant disparity in relative abundance: [evidence span withheld — anchor status: not_found]
- [readme] For local implementation using this GitHub repository, recent version of R (>4.2.0) is needed.: "For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
