---
name: time-period-lineage-trend-matching
description: Use when wastewater-derived mutation/lineage calls (e.g., lineagespot
  output from VCF files) must be validated against clinical genotype data sampled
  over the same area and multiple time periods, to judge whether the environmental
  surveillance signal aligns with clinical lineage dominance.
when_to_use_negative:
- No clinical genotypes from the same area and time periods exist, so no matched comparison
  can be made
- Input is raw FASTQ rather than variant calls — lineagespot operates on VCF files,
  not raw sequencing data
- Goal is absolute lineage frequency quantification rather than the qualitative predominant-lineage
  alignment assessment this skill implements
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: lineagespot
  role: R framework that infers SARS-CoV-2 lineage-related mutations from VCF files,
    providing the wastewater-side lineage assignments
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: Pangolin
  role: Defines the lineage assignment rules applied to patient-derived WGS samples
    used as the clinical reference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:c71ec1711ac6c3739f23c2b4b9b39f984b3b800544d28e26e8fc337b6a4dbb8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - lineagespot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# time-period-lineage-trend-matching

## Summary

Performs a qualitative, time-stratified comparison between SARS-CoV-2 lineages inferred by lineagespot from Thessaloniki municipal wastewater NGS samples and predominant lineages from Pangolin-assigned patient-derived WGS samples, across 14 time periods, to assess whether wastewater mutational surveillance tracks clinical lineage dynamics.

## When to use

Use when wastewater-derived mutation/lineage calls (e.g., lineagespot output from VCF files) must be validated against clinical genotype data sampled over the same area and multiple time periods, to judge whether the environmental surveillance signal aligns with clinical lineage dominance.

## When NOT to use

- No clinical genotypes from the same area and time periods exist, so no matched comparison can be made
- Input is raw FASTQ rather than variant calls — lineagespot operates on VCF files, not raw sequencing data
- Goal is absolute lineage frequency quantification rather than the qualitative predominant-lineage alignment assessment this skill implements

## Inputs

- lineagespot lineage assignments for 14 wastewater time periods (derived from freebayes VCF files)
- Pangolin-assigned patient-derived WGS lineage calls from Thessaloniki clinical samples (ENA PRJEB44141 / ERP128154)
- Time-period mapping between wastewater and clinical sampling dates

## Outputs

- Per-period comparison table (lineage_agreement.tsv) of predominant wastewater vs. predominant clinical lineage with agreement/disagreement flags
- Overall agreement rate across the 14 time periods

## How to apply

Load the lineagespot-derived lineage assignments for the 14 wastewater time periods (lineagespot is an R package whose inputs are VCF files) and the patient-derived lineage calls assigned under Pangolin lineage rules for the same period. For each of the 14 periods, tabulate the predominant wastewater lineage and the predominant clinical lineage from targeted clinical samples (ENA study PRJEB44141 / ERP128154), marking agreement or disagreement per period. Compute an overall agreement rate across the 14 periods to reproduce the paper's qualitative assessment of approximate alignment. Because wastewater variant calling used freebayes (judged the most informative caller for low-frequency variants < 0.01) and merged VCFs were collapsed at gene/amino-acid-substitution level to reduce noise, ensure the compared lineage labels derive from the same collapsed variant representation. Write the per-period comparison table with the agreement summary to lineage_agreement.tsv.

## Related tools

- **lineagespot** (R framework that infers SARS-CoV-2 lineage-related mutations from VCF files, providing the wastewater-side lineage assignments) — github.com/BiodataAnalysisGroup/lineagespot
- **Pangolin** (Defines the lineage assignment rules applied to patient-derived WGS samples used as the clinical reference)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- lineage_agreement.tsv contains exactly 14 rows (one per time period) plus a summary of the overall agreement rate
- Every period has both a wastewater predominant lineage and a clinical predominant lineage assigned (no unlabelled comparisons)
- The inferred lineages are only 'roughly aligned' to predominant clinical lineages per the paper — a claim of exact per-period agreement should trigger a review
- Wastewater lineage labels follow Pangolin lineage definitions (e.g., B.1.1.7), and the Alpha (B.1.1.7) evolution and domination should be visible across the timeline
- Variant calls underlying the wastewater side come from freebayes with low-frequency detection below 0.01, consistent with the paper's methodology

## Limitations

- Pangolin-based lineage assignment can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping to the reference genome were not applied for the four samples between 19 March 2021 and 15 April 2021

## Evidence

- [results] Qualitative assessment comparing major lineages in targeted clinical samples with wastewater data: "Qualitative assessment between the major lineages found in the targeted clinical samples, retrieved from the ENA study ID: PRJEB44141 (ERP128154), and across the 14 time periods"
- [intro] Validation across 14 time periods correlated with clinical lineages: "The method is tested and validated across 14 municipal wastewater samples retrieved in Thessaloniki, Greece in 14 different time periods"
- [abstract] Correlation of wastewater mutations with patient mutations from same area and periods: "allowed the correlation between the mutations evident through our approach and the mutations observed in patients from the same area and time periods"
- [readme] lineagespot operates on variant call files in R: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [intro] Pangolin lineage definitions anchor the method: "the lineage assignments defined by Pangolin13, this method identifies all SARS-CoV-2 variants present in the wastewater"
