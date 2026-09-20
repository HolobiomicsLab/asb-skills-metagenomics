---
name: lineage-mutation-profile-matching
description: Use when NGS variant calls from wastewater or clinical samples must be
  assigned to SARS-CoV-2 lineages and converted into per-sample lineage-abundance
  estimates to track variant dynamics over time (e.g., the rise of B.1.1.7).
when_to_use_negative:
- Input is raw FASTQ without prior variant calling — lineagespot requires variant(s)
  file(s) (VCF), not raw reads
- Goal is consensus whole-genome lineage assignment of a single high-coverage isolate
  — use Pangolin directly instead of wastewater read-profile matching
- Samples lack sufficient depth for low-frequency (< 0.01) variant detection, e.g.,
  heavily decayed wastewater RNA
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0199
tools:
- name: lineagespot
  role: R framework that identifies SARS-CoV-2 related mutations from VCF file(s)
    and infers lineage distributions against Pangolin/outbreak.info profiles
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language and environment for running lineagespot and building
    per-sample abundance tables
- name: Pangolin
  role: Source of lineage definitions used to match mutation profiles
- name: freebayes
  role: Variant caller producing the low-frequency (< 0.01) mutation calls that feed
    lineage matching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:8800cf47cb408c8b8fceb2b9fa53ddc091159431548e5c74bc0a560ffe166ac3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lineage-mutation-profile-matching

## Summary

A lineagespot-based R workflow that scores sample variant calls (VCF-derived amino acid substitutions) against lineage-characteristic mutation profiles (Pangolin rules / outbreak.info definitions) to infer SARS-CoV-2 lineage abundances in wastewater.

## When to use

Use when NGS variant calls from wastewater or clinical samples must be assigned to SARS-CoV-2 lineages and converted into per-sample lineage-abundance estimates to track variant dynamics over time (e.g., the rise of B.1.1.7).

## When NOT to use

- Input is raw FASTQ without prior variant calling — lineagespot requires variant(s) file(s) (VCF), not raw reads
- Goal is consensus whole-genome lineage assignment of a single high-coverage isolate — use Pangolin directly instead of wastewater read-profile matching
- Samples lack sufficient depth for low-frequency (< 0.01) variant detection, e.g., heavily decayed wastewater RNA

## Inputs

- VCF file(s) (single or a list) of called SARS-CoV-2 mutations from wastewater samples
- Merged, gene/amino-acid-collapsed mutation table with allele frequencies
- Pangolin lineage rules or outbreak.info lineage-characteristic mutation profiles

## Outputs

- Per-sample inferred lineage distribution/abundance table
- Time series of B.1.1.7 (Alpha) abundance across the 14 sampling time periods

## How to apply

Load the merged and collapsed mutation/lineage report (VCF files merged into a combined table of nucleotide mutations, amino acid substitutions, allele frequencies and overlapping genes, then collapsed at the gene and amino acid substitution level to reduce noise) into R. Apply lineagespot's lineage-abundance estimation, scoring each sample's reads against lineage-characteristic mutation profiles retrieved either as a Pangolin rule or through the outbreak.info definition. Assemble a per-sample table ranking the inferred lineage distributions, then extract the lineage-of-interest (e.g., B.1.1.7) abundance time series across sampling periods. freebayes was considered the most productive and informative variant calling tool for low-frequency variant detection (mutations down to < 0.01 frequency) feeding this step.

## Related tools

- **lineagespot** (R framework that identifies SARS-CoV-2 related mutations from VCF file(s) and infers lineage distributions against Pangolin/outbreak.info profiles) — github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language and environment for running lineagespot and building per-sample abundance tables)
- **Pangolin** (Source of lineage definitions used to match mutation profiles)
- **freebayes** (Variant caller producing the low-frequency (< 0.01) mutation calls that feed lineage matching)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); library(lineagespot)  # then run lineagespot on the merged VCF list from the 14 wastewater samples
```

## Evaluation signals

- The extracted B.1.1.7 (Alpha) abundance fraction rises over successive sampling time periods across the 6-month window, recording its rapid domination
- Per-sample abundance table covers all 14 Thessaloniki time periods with plausible lineage rankings
- Merged mutation table contains nucleotide mutations with corresponding amino acid substitutions, allele frequency and overlapping gene annotations
- Collapsed table shows one entry per gene and amino acid substitution (noise from nucleotide mutations mapping to the same amino acid change removed)
- Detected mutations correlate with mutations observed in patients from the same area and time periods

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage (rule A vs rule B both mapping to lineage L)
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: for four samples between 19 March 2021 and 15 April 2021 primer trimming and remapping to the reference genome were not applied
- No software versions reported for Minimap2, iVar, freebayes, or SnpEff, complicating reproducibility

## Evidence

- [abstract] lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community: "lineagespot was able to record the evolution and rapid domination of the Alpha variant (B.1.1.7) in the community"
- [results] lineage definition from Pangolin rule or outbreak.info: "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [results] Table 4 collapsed at gene and amino acid substitution level: "Table 4 was collapsed at the gene and amino acid substitution level, therefore reducing any data noise that is introduced by nucleotide mutations"
- [readme] lineagespot identifies mutations from VCF files: "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s) (i.e., [variant calling format]"
- [discussion] reads satisfying different rules of the same lineage cause incorrect assignment: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
