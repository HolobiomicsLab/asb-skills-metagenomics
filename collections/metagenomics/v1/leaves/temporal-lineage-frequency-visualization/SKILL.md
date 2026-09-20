---
name: temporal-lineage-frequency-visualization
description: Use when NGS wastewater samples collected over successive time periods
  have been assigned lineages (e.g. via lineagespot/Pangolin rules) and the goal is
  to rank per-sample lineage abundances and plot a variant's temporal trajectory.
when_to_use_negative:
- Input is raw FASTQ reads rather than variant-called VCF/lineage tables — run freebayes
  variant calling and the lineagespot merge/collapse steps first
- Samples are single clinical isolates without a time-series design, so no temporal
  trajectory can be assembled
- Only a single sample/time point is available, leaving no successive periods to verify
  a rising fraction
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0085
tools:
- name: lineagespot
  role: R framework that merges variant calls, collapses mutations to amino-acid level,
    scores reads against lineage mutation profiles, and yields per-sample lineage
    abundances for the time series
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language of lineagespot; used to load reports, assemble the
    abundance table, and visualize the B.1.1.7 trajectory
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: Pangolin
  role: Source of lineage definitions/rules used to assign reads to lineages when
    scoring per-sample abundances
- name: freebayes
  role: Variant caller of choice producing the low-frequency (< 0.01) mutation VCFs
    that feed the lineage-abundance estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:d0440ce3e88b8e5206697a1e459d5105adcde670f6e8e52dd1eff2f62a3f09e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# temporal-lineage-frequency-visualization

## Summary

Assemble a per-sample lineage-abundance table from lineagespot output across 14 Thessaloniki wastewater time periods and extract/visualize the B.1.1.7 (Alpha) abundance time series to capture the variant's rise over the 6-month window.

## When to use

Use when NGS wastewater samples collected over successive time periods have been assigned lineages (e.g. via lineagespot/Pangolin rules) and the goal is to rank per-sample lineage abundances and plot a variant's temporal trajectory.

## When NOT to use

- Input is raw FASTQ reads rather than variant-called VCF/lineage tables — run freebayes variant calling and the lineagespot merge/collapse steps first
- Samples are single clinical isolates without a time-series design, so no temporal trajectory can be assembled
- Only a single sample/time point is available, leaving no successive periods to verify a rising fraction

## Inputs

- Merged and collapsed lineagespot lineage/abundance reports (VCF-derived tables with nucleotide mutations, amino acid substitutions, allele frequencies) for the 14 wastewater time-point samples
- Lineage-characteristic mutation profiles (Pangolin rules or outbreak.info pre-calculated definitions)
- Sample time-period metadata for the 14 sampling dates covering the 6-month period

## Outputs

- Per-sample lineage-abundance table ranking inferred SARS-CoV-2 lineage distributions across the 14 time periods
- B.1.1.7 (Alpha) abundance time series extracted from the table
- Trajectory plot of B.1.1.7 prevalence over the 6-month window showing its evolution and rapid domination

## How to apply

Load the merged, collapsed lineage report (amino-acid substitution level, after VCF merging and gene/aa collapse) for all 14 wastewater time-point samples into R. Score each sample's reads against lineage-characteristic mutation profiles (Pangolin rules or outbreak.info definitions) to obtain per-sample inferred lineage distributions. Assemble a per-sample abundance table ranking lineage fractions across the 14 time periods, then extract the B.1.1.7 (Alpha) column as a time series and plot its trajectory over the 6-month window. Validate that the B.1.1.7 fraction rises over successive time periods, reproducing the reported rapid domination of Alpha in the community.

## Related tools

- **lineagespot** (R framework that merges variant calls, collapses mutations to amino-acid level, scores reads against lineage mutation profiles, and yields per-sample lineage abundances for the time series) — github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language of lineagespot; used to load reports, assemble the abundance table, and visualize the B.1.1.7 trajectory) — https://github.com/BiodataAnalysisGroup/lineagespot
- **Pangolin** (Source of lineage definitions/rules used to assign reads to lineages when scoring per-sample abundances)
- **freebayes** (Variant caller of choice producing the low-frequency (< 0.01) mutation VCFs that feed the lineage-abundance estimation)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); lineagespot::lineagespot(vcf_folder = "merged_vcfs/", af = 0.01)  # then extract B.1.1.7 per-sample abundances in R
```

## Evaluation signals

- The per-sample abundance table contains a ranked lineage distribution for each of the 14 time periods covering the 6-month window
- The extracted B.1.1.7 (Alpha) fraction increases monotonically or near-monotonically over successive time periods, reproducing the reported rapid domination
- Each sample's lineage assignment is derived from lineage-characteristic mutation profiles (Pangolin rules or outbreak.info definitions) applied to collapsed amino-acid substitutions, not raw nucleotide VCFs
- Qualitative check: dominant lineages in the table agree with targeted clinical samples from the same ENA study (PRJEB44141/ERP128154) time periods
- The trajectory plot's x-axis spans the full sampling period (e.g. early February 2021 to mid-April 2021+) matching the 14 sampling periods

## Limitations

- Pangolin-based lineage assignment assumes reads satisfying any one rule of lineage L imply lineage L, which can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across the series: for the four samples between 19 March 2021 and 15 April 2021 primer trimming and remapping to the reference genome were not applied
- Wastewater abundances are inferred distributions, not exact strain composition estimates

## Evidence

- [abstract] lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community: "lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community"
- [results] the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition): "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [results] Qualitative assessment between the major lineages found in the targeted clinical samples: "Qualitative assessment between the major lineages found in the targeted clinical samples, retrieved from the ENA study ID: PRJEB44141 (ERP128154), and across the 14 time periods"
- [results] Table 4 was collapsed at the gene and amino acid substitution level, therefore reducing any data noise: "Table 4 was collapsed at the gene and amino acid substitution level, therefore reducing any data noise that is introduced by nucleotide mutations that correspond to the same amino acid change"
- [discussion] reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
