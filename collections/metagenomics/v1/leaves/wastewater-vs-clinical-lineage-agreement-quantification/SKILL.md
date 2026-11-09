---
name: wastewater-vs-clinical-lineage-agreement-quantification
description: Use when wastewater-inferred lineage calls (e.g., from lineagespot on
  VCF inputs) must be compared against clinical WGS lineage assignments from the same
  area and time windows to assess surveillance agreement.
when_to_use_negative:
- The goal is exact lineage-frequency reconstruction per sample rather than qualitative
  agreement with predominant clinical lineages — the paper's assessment is explicitly
  qualitative
- Clinical lineage calls for the same area and time periods are unavailable, so no
  matched comparison can be made
- Input variant calls were not produced under the lineagespot methodology (e.g., Pangolin
  run directly on wastewater consensus genomes), as the Pangolin-rule matching on
  wastewater reads can mis-assign lineages
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0085
tools:
- name: lineagespot
  role: R framework that identifies SARS-CoV-2 lineages/mutations from VCF files,
    producing the wastewater lineage assignments to be compared
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: Pangolin
  role: Defines the lineage assignment rules used for the patient-derived clinical
    WGS lineage calls
- name: R
  role: Implementation language in which the lineagespot package runs and in which
    the per-period agreement table is computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:c60f7df470786263233af253599b31852a990bd741fea691aed987c15abdd1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - lineagespot
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# wastewater-vs-clinical-lineage-agreement-quantification

## Summary

A validation skill that quantifies how well SARS-CoV-2 lineages inferred by lineagespot from wastewater samples align with predominant patient-derived lineages (Pangolin-assigned) across matched time periods, reproducing the paper's qualitative assessment over 14 Thessaloniki sampling periods.

## When to use

Use when wastewater-inferred lineage calls (e.g., from lineagespot on VCF inputs) must be compared against clinical WGS lineage assignments from the same area and time windows to assess surveillance agreement.

## When NOT to use

- The goal is exact lineage-frequency reconstruction per sample rather than qualitative agreement with predominant clinical lineages — the paper's assessment is explicitly qualitative
- Clinical lineage calls for the same area and time periods are unavailable, so no matched comparison can be made
- Input variant calls were not produced under the lineagespot methodology (e.g., Pangolin run directly on wastewater consensus genomes), as the Pangolin-rule matching on wastewater reads can mis-assign lineages

## Inputs

- lineagespot-derived lineage assignments for 14 wastewater time periods (R data frames from the lineagespot package)
- patient-derived WGS lineage calls for Thessaloniki (ENA study PRJEB44141 / ERP128141 clinical samples) assigned under Pangolin lineage rules
- time-period mapping (dates) aligning wastewater samples to clinical sampling windows

## Outputs

- Per-period comparison table of predominant wastewater lineage vs predominant clinical lineage with agreement/disagreement flags (lineage_agreement.tsv)
- Overall agreement rate across the 14 time periods

## How to apply

Load the lineagespot-derived lineage assignments for each wastewater time period in R (lineagespot package, outputs per its User Guide), then load the patient-derived WGS lineage calls for the same area, defined by Pangolin lineage assignment rules. For each of the 14 time periods (e.g., the 05-11 February 2021 period used for variant-caller benchmarking), tabulate the predominant wastewater lineage and the predominant clinical lineage, marking agreement or disagreement per period. Compute an overall agreement rate across the 14 periods, which in the paper supports a qualitative finding of approximate alignment rather than exact per-period matching. Rationale: the paper found lineagespot recorded the evolution and rapid domination of the Alpha variant (B.1.1.7) and that wastewater mutations correlated with mutations observed in patients from the same area and time periods, so agreement is expected to be approximate and dominated by major variants like B.1.1.7.

## Related tools

- **lineagespot** (R framework that identifies SARS-CoV-2 lineages/mutations from VCF files, producing the wastewater lineage assignments to be compared) — github.com/BiodataAnalysisGroup/lineagespot
- **Pangolin** (Defines the lineage assignment rules used for the patient-derived clinical WGS lineage calls)
- **R** (Implementation language in which the lineagespot package runs and in which the per-period agreement table is computed)

## Examples

```
Rscript -e 'library(lineagespot); ww <- read.csv("lineagespot_lineages.tsv", sep="\t"); clin <- read.csv("pangolin_clinical.tsv", sep="\t"); agree <- ww$predominant_lineage == clin$predominant_lineage; write.csv(data.frame(period=ww$period, ww_lineage=ww$predominant_lineage, clin_lineage=clin$predominant_lineage, agree=agree), "lineage_agreement.tsv", quote=FALSE)'
```

## Evaluation signals

- The output lineage_agreement.tsv contains exactly 14 rows, one per wastewater time period, each with a predominant wastewater lineage, predominant clinical lineage, and an agreement flag
- Predominant clinical lineages are drawn from Pangolin-assigned calls (e.g., B.1.1.7 dominating in spring 2021), and the wastewater calls show the same evolution and rapid domination of Alpha (B.1.1.7)
- Overall agreement rate is computed as the fraction of the 14 periods with matching predominant lineages, and is reported as approximate alignment consistent with the paper's qualitative finding
- Wastewater-inferred mutations correlate with mutations observed in patients from the same area and time periods, providing a secondary consistency check
- Lineage rules used for wastewater assignment match Pangolin or outbreak.info lineage definitions

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: for the four samples between 19 March 2021 and 15 April 2021 primer trimming and remapping to the reference genome was not applied

## Evidence

- [results] Qualitative assessment of wastewater vs clinical lineage agreement: "Qualitative assessment between the major lineages found in the targeted clinical samples, retrieved from the ENA study ID: PRJEB44141 (ERP128154), and across the 14 time periods"
- [abstract] Alpha domination recorded in wastewater: "lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community"
- [abstract] Correlation with patient mutations: "allowed the correlation between the mutations evident through our approach and the mutations observed in patients from the same area and time periods"
- [discussion] Pangolin rule mis-assignment caveat: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
- [readme] lineagespot is an R framework for VCF-based lineage detection: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
