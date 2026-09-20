---
name: zero-depth-position-filtering
description: Use when building per-lineage rule vectors from SARS-CoV-2 VCF calls
  against NC_045512 in lineagespot, where positions in the reference-base portion
  of the rule vector must be dropped if no reads cover that reference position.
when_to_use_negative:
- The input VCF already excludes zero-coverage positions, so re-filtering would be
  redundant
- Working with consensus-level clinical genome calls rather than low-frequency wastewater
  VCFs, where rule-satisfaction proportions are not the analysis target
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: lineagespot
  role: R framework that builds the Pangolin rule vectors and applies the zero-depth
    position removal within lineage rule evaluation
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language for the rule-vector construction and filtering logic
- name: data.table
  role: Fast tabular reading of VCF calls and NC_045512 reference positions in R
- name: stringr
  role: Splitting comma-delimited multi-allelic ALT fields into single-ALT rules during
    vector construction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:d09e699b351f27669515f924abdc6337cb1722642682d8ebf8427fc6e8fd2c03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - data.table
  - lineagespot
  - stringr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# zero-depth-position-filtering

## Summary

A filtering step within the lineagespot Pangolin rule-vector expansion that removes positions whose reference read depth equals zero from the first vector of the lineage rule set, so that uninformative reference positions do not count against lineage-rule satisfaction.

## When to use

Use when building per-lineage rule vectors from SARS-CoV-2 VCF calls against NC_045512 in lineagespot, where positions in the reference-base portion of the rule vector must be dropped if no reads cover that reference position.

## When NOT to use

- The input VCF already excludes zero-coverage positions, so re-filtering would be redundant
- Working with consensus-level clinical genome calls rather than low-frequency wastewater VCFs, where rule-satisfaction proportions are not the analysis target

## Inputs

- VCF file(s) of SARS-CoV-2 variant calls from wastewater NGS data (e.g., freebayes output)
- Pangolin lineage rule definitions against the NC_045512 (Wuhan) reference genome
- Per-lineage rule vectors with reference base and ALT rule entries

## Outputs

- Filtered per-lineage rule vector (CSV) with zero-depth reference positions removed
- Rule-satisfaction tallies (number of satisfied variants vs number of lineage rules) per lineage per sample

## How to apply

After constructing the per-lineage rule vector in R (reference base at each defining Pangolin position, followed by four appended rules requiring an observed base that differs from that reference base), scan the first vector and remove any position whose reference read depth is zero, since absence of coverage makes the reference-base rule unverifiable rather than unsatisfied. Read the VCF mutation calls and NC_045512 reference positions with data.table, and split comma-delimited multi-allelic ALT fields into separate single-ALT rules with stringr before tallying rule satisfaction. The rationale is that zero-depth positions would otherwise inflate the denominator of lineage rules and bias the proportion of satisfied rules used to compare against the Pangolin lineage report.

## Related tools

- **lineagespot** (R framework that builds the Pangolin rule vectors and applies the zero-depth position removal within lineage rule evaluation) — github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language for the rule-vector construction and filtering logic)
- **data.table** (Fast tabular reading of VCF calls and NC_045512 reference positions in R)
- **stringr** (Splitting comma-delimited multi-allelic ALT fields into single-ALT rules during vector construction)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- Output CSV exists and contains per-lineage rule entries with position, reference base, and appended ALT rules
- No position with zero reference read depth remains in the first vector of the rule set
- Comma-delimited multi-allelic ALT entries appear as separate single-ALT rules after filtering
- Rule-satisfaction proportions computed on the filtered vector reproduce lineage assignments consistent with the Pangolin lineage report

## Limitations

- Lineage assignment can be incorrect when different groups of reads satisfy different rules of the same lineage L
- Preprocessing inconsistencies (e.g., primer trimming and remapping omitted for four samples between 19 March and 15 April 2021) can affect coverage and thus which positions have zero depth
- Viral adsorption and decay in wastewater may hinder mutation identification and coverage at defining positions

## Evidence

- [discussion] filter rationale: "positions with reference read depth equal to zero are removed from the first vector"
- [results] lineage rule source: "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [abstract] tool implementation: "lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools"
