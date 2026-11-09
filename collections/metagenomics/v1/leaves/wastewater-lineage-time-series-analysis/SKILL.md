---
name: wastewater-lineage-time-series-analysis
description: Use when you have variant calls (VCFs) from multiple sequenced wastewater
  time-point samples and need to infer per-sample lineage distributions and visualize
  how one lineage's abundance changes over time.
when_to_use_negative:
- You need sample-level consensus genomes or formal Pangolin lineage calls from single
  clinical isolates — this method scores mixed wastewater read populations against
  lineage rules and can mis-assign when different reads satisfy different rules of
  the same lineage
- Input is raw FASTQ with no variant calls yet — run the variant-calling pipeline
  (freebayes against NC_045512) first; lineagespot consumes VCF-derived mutation tables,
  not reads directly
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0769
tools:
- name: lineagespot
  role: Core R framework that scores merged/collapsed VCF mutation tables against
    lineage-characteristic mutation profiles to infer per-sample lineage distributions
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: Pangolin
  role: Source of lineage definitions/rules against which wastewater mutations are
    matched for lineage-abundance scoring
- name: freebayes
  role: Variant caller of choice for detecting low-frequency (< 0.01) mutations in
    wastewater samples prior to lineage scoring
- name: R
  role: Implementation language; used to load lineage reports, assemble the per-sample
    abundance table, and visualize the B.1.1.7 time series
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:3d8c39b3e6ef8b52b30e4df5ad9bdab577c9e93d8eee7ce111061a6c8673c480
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - Pangolin
  - R
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# wastewater-lineage-time-series-analysis

## Summary

Track the rise and domination of a specific SARS-CoV-2 variant of concern (e.g., B.1.1.7/Alpha) across a time-series of wastewater samples by scoring sample-level variant calls against lineage-characteristic mutation profiles (Pangolin rules / outbreak.info definitions) and extracting the variant's per-sample abundance trajectory.

## When to use

Use when you have variant calls (VCFs) from multiple sequenced wastewater time-point samples and need to infer per-sample lineage distributions and visualize how one lineage's abundance changes over time.

## When NOT to use

- You need sample-level consensus genomes or formal Pangolin lineage calls from single clinical isolates — this method scores mixed wastewater read populations against lineage rules and can mis-assign when different reads satisfy different rules of the same lineage
- Input is raw FASTQ with no variant calls yet — run the variant-calling pipeline (freebayes against NC_045512) first; lineagespot consumes VCF-derived mutation tables, not reads directly

## Inputs

- Merged VCF-derived table of detected nucleotide mutations with amino acid substitutions, allele frequency, and overlapping gene per wastewater sample
- Collapsed (gene- and amino-acid-substitution-level) mutation table per sample to reduce noise from nucleotide mutations mapping to the same amino acid change
- Lineage-characteristic mutation profiles (Pangolin rules or outbreak.info definitions)
- Time-point metadata for the wastewater samples (e.g., 14 sampling periods over 6 months)

## Outputs

- Per-sample lineage-abundance table ranking inferred SARS-CoV-2 lineage distributions for each time period
- Time series of the target lineage (B.1.1.7) abundance across the sampling periods
- Trajectory plot showing the lineage's rise/domination over the study window

## How to apply

Run lineagespot (an R package) on the merged, collapsed variant-call tables: each sample's low-frequency nucleotide mutations and amino acid substitutions are scored against lineage-characteristic mutation profiles retrieved either as a Pangolin rule or through the outbreak.info definition, producing a per-sample abundance table ranking inferred lineage distributions. Load the merged and collapsed lineage reports for all time-point samples (here, 14 wastewater samples from Thessaloniki covering a 6-month period) into R, assemble a per-sample lineage-abundance matrix ordered by sampling period, then extract the target lineage's (B.1.1.7) abundance column as a time series and plot its trajectory. The variant-caller choice matters: freebayes was judged the most productive and informative for low-frequency variant detection (mutations below 0.01 allele frequency). Validate by checking that the extracted time series reproduces the expected epidemiological pattern — in the Thessaloniki use-case, the B.1.1.7 fraction rises over successive time periods until it dominates.

## Related tools

- **lineagespot** (Core R framework that scores merged/collapsed VCF mutation tables against lineage-characteristic mutation profiles to infer per-sample lineage distributions) — https://github.com/BiodataAnalysisGroup/lineagespot
- **Pangolin** (Source of lineage definitions/rules against which wastewater mutations are matched for lineage-abundance scoring)
- **freebayes** (Variant caller of choice for detecting low-frequency (< 0.01) mutations in wastewater samples prior to lineage scoring)
- **R** (Implementation language; used to load lineage reports, assemble the per-sample abundance table, and visualize the B.1.1.7 time series) — https://github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); report <- lineagespot(vcf_folder = "merged_vcf/", outbreak_info = TRUE); plot(lineagespot$B.1.1.7_abundance ~ sampling_period)
```

## Evaluation signals

- The B.1.1.7 (Alpha) abundance column of the per-sample table increases monotonically (or near-monotonically) over successive sampling periods, reproducing the reported 'evolution and rapid domination' of Alpha in the Thessaloniki community
- Each sample's lineage assignment table is populated for all 14 time periods with no missing samples after merging and collapsing VCFs
- Collapsed tables contain no redundant rows for distinct nucleotide mutations corresponding to the same amino acid substitution
- Lineage matches trace back to explicit Pangolin-rule or outbreak.info lineage-characteristic mutation profiles, not ad-hoc mutation lists
- Qualitative consistency with major lineages found in targeted clinical samples from the same area and time periods (ENA study PRJEB44141/ERP128154)

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different groups of reads satisfy different rules (rule A and rule B) of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping to the reference genome were omitted for the four samples between 19 March 2021 and 15 April 2021
- No software versions were reported for Minimap2, iVar, freebayes, or SnpEff, complicating exact reproduction

## Evidence

- [abstract] lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community: "lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community"
- [results] the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition): "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [results] we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned.: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
- [readme] Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s) (i.e., [variant calling format]: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s) (i.e., [variant calling"
