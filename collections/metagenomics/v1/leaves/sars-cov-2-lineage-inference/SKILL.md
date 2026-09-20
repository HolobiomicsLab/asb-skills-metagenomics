---
name: sars-cov-2-lineage-inference
description: Use when you have VCF files of low-frequency SARS-CoV-2 variants (e.g.,
  from ARTIC-amplicon wastewater sequencing) and need to detect which lineages are
  present and estimate their relative abundance over time.
when_to_use_negative:
- Input reads have not been mapped/variant-called — lineagespot consumes VCF files,
  not raw FASTQ; run the ARTIC-style variant calling pipeline first
- The goal is Pangolin-style whole-genome consensus lineage assignment of a single
  high-coverage clinical genome; the paper notes Pangolin-rule-based assignment on
  read groups can misassign lineages when different reads satisfy different rules
  of the same lineage
- Viral target is not SARS-CoV-2 or the reference genome is not NC_045512 — the mutation-to-lineage
  matching depends on SARS-CoV-2-specific profiles
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0080
tools:
- name: lineagespot
  role: Core R framework that takes VCF files plus GFF3 annotation and reference genome,
    matches mutations to lineage profiles, and returns variant, lineage-hit, and lineage-report
    tables
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language/runtime in which lineagespot is installed and executed
- name: freebayes
  role: Variant caller used upstream to produce the low-frequency (< 0.01) VCF inputs;
    assessed as the most productive and informative of the three callers tested
- name: Minimap2
  role: Maps cleaned paired-end reads to the NC_045512 SARS-CoV-2 reference genome
    with minimal chaining score 40 before variant calling
- name: Pangolin
  role: Source of lineage definitions used to score mutations against lineage-characteristic
    profiles
- name: SnpEff
  role: Annotates variants using the NC_045512.2 (version 5.0) database
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:7beeeae2f7b7bf3da717bc64fcf32b8779207f5c2189b74ee5b522974bd506a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Minimap2
  - Pangolin
  - R
  - SnpEff
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sars-cov-2-lineage-inference

## Summary

Infer SARS-CoV-2 lineage composition and per-lineage abundance metrics from wastewater NGS variant calls using the lineagespot R package, which matches detected mutations against lineage-characteristic mutation profiles (Pangolin rules / outbreak.info definitions).

## When to use

Use when you have VCF files of low-frequency SARS-CoV-2 variants (e.g., from ARTIC-amplicon wastewater sequencing) and need to detect which lineages are present and estimate their relative abundance over time.

## When NOT to use

- Input reads have not been mapped/variant-called — lineagespot consumes VCF files, not raw FASTQ; run the ARTIC-style variant calling pipeline first
- The goal is Pangolin-style whole-genome consensus lineage assignment of a single high-coverage clinical genome; the paper notes Pangolin-rule-based assignment on read groups can misassign lineages when different reads satisfy different rules of the same lineage
- Viral target is not SARS-CoV-2 or the reference genome is not NC_045512 — the mutation-to-lineage matching depends on SARS-CoV-2-specific profiles

## Inputs

- VCF files (or folder of VCFs) of variant calls from wastewater NGS data, ideally from low-frequency-aware calling (freebayes)
- NC_045512.2 GFF3 annotation file
- SARS-CoV-2 Wuhan-Hu-1 reference genome FASTA (NC_045512)
- Lineage-characteristic mutation profiles (Pangolin rules or outbreak.info definitions)
- Optionally: cleaned FASTQ reads (paired-end, >= 200 nt after trimming with sliding window quality threshold 15)

## Outputs

- Overall variant table: all detected nucleotide mutations with amino-acid substitutions, allele frequencies, and overlapping gene
- Lineage hits table: mutations assigned to SARS-CoV-2 lineages
- Lineage report table: per-lineage abundance metrics (mean AF, mean unique AF, non-zero min AF, variant counts and proportions)
- Per-sample lineage abundance time series (e.g., B.1.1.7 fraction across sampling periods)

## How to apply

Map cleaned paired-end reads to the SARS-CoV-2 Wuhan reference (NC_045512.2) with Minimap2 (minimal chaining score 40), call variants with freebayes (assessed as the most productive caller for low-frequency variants, < 0.01), and annotate with SnpEff using the NC_045512.2 (version 5.0) database. In R, install lineagespot via devtools from GitHub (BiodataAnalysisGroup/lineagespot) and call the single lineagespot() function, supplying the VCF folder, the NC_045512.2 GFF3 annotation, and the Wuhan reference FASTA so detected nucleotide mutations are matched to amino-acid substitutions and then to lineage-characteristic mutation profiles retrieved as Pangolin rules or outbreak.info definitions. The function returns three tables: an overall variant table (nucleotide mutations, amino-acid substitutions, allele frequencies), a lineage hits table, and a lineage report with per-lineage mean AF, mean unique AF, non-zero min AF, and variant counts/proportions. For time-series work (e.g., 14 Thessaloniki wastewater samples over 6 months), collapse mutation assignments at the gene and amino-acid substitution level to reduce noise, then extract per-lineage abundance trajectories such as the B.1.1.7 (Alpha) fraction. Write all three tables to CSV and check that the lineage report contains the expected per-lineage summary fields.

## Related tools

- **lineagespot** (Core R framework that takes VCF files plus GFF3 annotation and reference genome, matches mutations to lineage profiles, and returns variant, lineage-hit, and lineage-report tables) — github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language/runtime in which lineagespot is installed and executed)
- **freebayes** (Variant caller used upstream to produce the low-frequency (< 0.01) VCF inputs; assessed as the most productive and informative of the three callers tested)
- **Minimap2** (Maps cleaned paired-end reads to the NC_045512 SARS-CoV-2 reference genome with minimal chaining score 40 before variant calling)
- **Pangolin** (Source of lineage definitions used to score mutations against lineage-characteristic profiles)
- **SnpEff** (Annotates variants using the NC_045512.2 (version 5.0) database)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); tables <- lineagespot(vcf_folder = "vcf/", gff3 = "NC_045512.2.gff3", ref = "NC_045512.2.fasta")
```

## Evaluation signals

- All three output tables (overall variant table, lineage hits, lineage report) exist and are non-empty after the lineagespot() call
- The lineage report contains the expected per-lineage summary fields: mean AF, mean unique AF, non-zero min AF, variant counts and proportions
- Variant table entries carry amino-acid substitutions and allele frequencies consistent with the annotated (SnpEff NC_045512.2 v5.0) input VCFs
- For the 14 Thessaloniki wastewater time points, the B.1.1.7 (Alpha) abundance series rises over successive time periods, reproducing the reported rapid domination
- Allele-frequency replicates agree: Student's t-test on mean AF and F-test on variances show no significant difference between replicate VCF pairs

## Limitations

- Pangolin-rule-based assignment assumes all rule-satisfying reads belong to one lineage; if one read group satisfies rule A of lineage L and another satisfies rule B of the same lineage L, lineage L is incorrectly assigned
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: for four samples between 19 March 2021 and 15 April 2021, primer trimming and remapping to the reference genome were not applied
- Only paired-end sequences are retained; unmatched and multiple mappings are removed, and reads must be >= 200 nt after trimming (sliding window quality 15, width 4)

## Evidence

- [abstract] lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community: "lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community"
- [results] we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [results] lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition): "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [discussion] mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [readme] lineagespot is a framework written in R, and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s): "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
