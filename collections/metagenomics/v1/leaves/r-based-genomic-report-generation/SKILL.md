---
name: r-based-genomic-report-generation
description: Use when you have one or more SARS-CoV-2 VCF files from NGS sequencing
  of wastewater (or clinical) samples and need to assign lineage-characteristic mutations
  and infer lineage distributions over sampling time periods.
when_to_use_negative:
- Input is raw FASTQ rather than variant calls — preprocessing (trimming to minimum
  200 nt with sliding window quality 15, read filtering) and variant calling must
  be done first via the raw-data-analysis pipeline
- Single-genome clinical isolate typing requiring a definitive consensus lineage call
  rather than a mixed community lineage distribution
- Non-SARS-CoV-2 pathogens — the tool is built around Pangolin/outbreak.info SARS-CoV-2
  lineage definitions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: lineagespot
  role: R framework that ingests VCF file(s), merges and collapses mutations, and
    scores reads against Pangolin/outbreak.info lineage profiles to infer lineage
    distributions
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: freebayes
  role: Variant caller upstream of report generation; assessed as the most productive
    and informative option for low-frequency (< 0.01) variant detection in wastewater
- name: Pangolin
  role: Source of lineage rules used by lineagespot to assign reads to lineages
- name: R
  role: Implementation language; the skill is exercised as an R package workflow
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:caa2d6844fcc6f09b3be7d1f38163cd04eee0d7289887685f5cdaeb04cdef7db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - Pangolin
  - R
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# r-based-genomic-report-generation

## Summary

Generates merged, collapsed lineage-abundance reports from SARS-CoV-2 variant call format (VCF) files using the R package lineagespot, enabling monitoring of mutations and detection of lineages (e.g., Alpha/B.1.1.7) across wastewater time-series samples.

## When to use

Use when you have one or more SARS-CoV-2 VCF files from NGS sequencing of wastewater (or clinical) samples and need to assign lineage-characteristic mutations and infer lineage distributions over sampling time periods.

## When NOT to use

- Input is raw FASTQ rather than variant calls — preprocessing (trimming to minimum 200 nt with sliding window quality 15, read filtering) and variant calling must be done first via the raw-data-analysis pipeline
- Single-genome clinical isolate typing requiring a definitive consensus lineage call rather than a mixed community lineage distribution
- Non-SARS-CoV-2 pathogens — the tool is built around Pangolin/outbreak.info SARS-CoV-2 lineage definitions

## Inputs

- One or more VCF (variant calling format) files from SARS-CoV-2 variant calling (e.g., freebayes output)
- FASTQ-derived variants mapped to the SARS-CoV-2 Wuhan reference genome NC_045512
- Lineage definitions (Pangolin rules or outbreak.info lineage-characteristic mutation profiles)

## Outputs

- Merged table of nucleotide mutations with amino acid substitutions, allele frequency, and overlapping gene
- Collapsed gene/amino-acid-level mutation table
- Per-sample SARS-CoV-2 lineage abundance table (e.g., B.1.1.7 fraction per time period)
- Lineage time-series visualization (e.g., Alpha variant trajectory across 14 sampling periods)

## How to apply

Install the package with devtools::install_github("BiodataAnalysisGroup/lineagespot"), then supply a single VCF or a list of VCF files derived from variant calling of reads mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512). The tool merges all VCF files into a combined table of detected nucleotide mutations with corresponding amino acid substitutions, allele frequencies, and overlapping genes, then collapses the table at the gene and amino acid substitution level to reduce noise from nucleotide mutations mapping to the same amino acid change. Lineage scoring is performed against lineage definitions retrieved as Pangolin rules or outbreak.info definitions, yielding per-sample lineage distributions that can be assembled into a per-time-period abundance table (e.g., 14 Thessaloniki wastewater samples spanning 6 months) and visualized as a time series. Correct application is judged by whether the resulting abundance table and plots recover known dynamics, such as the rise and rapid domination of B.1.1.7 in successive sampling periods.

## Related tools

- **lineagespot** (R framework that ingests VCF file(s), merges and collapses mutations, and scores reads against Pangolin/outbreak.info lineage profiles to infer lineage distributions) — https://github.com/BiodataAnalysisGroup/lineagespot
- **freebayes** (Variant caller upstream of report generation; assessed as the most productive and informative option for low-frequency (< 0.01) variant detection in wastewater)
- **Pangolin** (Source of lineage rules used by lineagespot to assign reads to lineages)
- **R** (Implementation language; the skill is exercised as an R package workflow) — https://github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); lineagespot(vcf = list_of_14_vcf_files, ref = "NC_045512")
```

## Evaluation signals

- The merged output table contains nucleotide mutations, amino acid substitutions, allele frequency, and overlapping gene columns for every input VCF sample
- Collapsing at the gene/amino-acid level removes redundant nucleotide-level entries corresponding to the same amino acid change
- For time-series input (14 Thessaloniki samples over 6 months), the B.1.1.7 abundance fraction rises across successive sampling periods, reflecting its recorded rapid domination
- Lineage assignments are consistent with the qualitative assessment against patient-derived lineages from ENA study PRJEB44141 (ERP128154) for the same time periods

## Limitations

- Pangolin-rule-based assignment can misassign when different read groups satisfy different rules of the same lineage (rule A vs rule B of lineage L both yield lineage L incorrectly)
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping to the reference genome were omitted for four samples between 19 March 2021 and 15 April 2021
- No software versions reported for Minimap2, iVar, freebayes, or SnpEff, complicating exact reproduction

## Evidence

- [abstract] lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools: "lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools"
- [abstract] lineagespot recorded Alpha domination: "lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community"
- [results] VCF merge description: "all VCF files were merged into a combined table in which all the detected nucleotide mutations along with the corresponding amino acid substitutions that have been identified are stored"
- [results] Collapse rationale: "Table 4 was collapsed at the gene and amino acid substitution level, therefore reducing any data noise that is introduced by nucleotide mutations that correspond to the same amino acid change"
- [discussion] Lineage assignment failure mode: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
- [readme] README description: "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s) (i.e., [variant calling format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531692-VCF-Var"
