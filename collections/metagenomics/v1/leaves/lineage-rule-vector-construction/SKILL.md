---
name: lineage-rule-vector-construction
description: Use when you need to convert a SARS-CoV-2 lineage definition (Pangolin
  rule or outbreak.info profile) into an evaluable rule vector against the NC_045512
  reference, so VCF-derived variants can be scored for rule satisfaction per lineage.
when_to_use_negative:
- Input is not variant call data (e.g., raw FASTQ reads) — reads must first be mapped
  to NC_045512 with Minimap2 and variant-called (e.g., freebayes) before rule evaluation
- The goal is whole-genome phylogenetic placement of a single high-coverage consensus
  genome, where standard Pangolin on a consensus FASTA is the established route rather
  than rule-satisfaction scoring of low-frequency wastewater variants
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: lineagespot
  role: R framework that builds the lineage rule vectors and scores variants against
    them
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: data.table
  role: Fast tabular reading of VCF calls and reference positions in R
- name: stringr
  role: Splits comma-delimited multi-allelic ALT fields into separate single-ALT rules
- name: Pangolin
  role: Source of lineage definitions/rules used to construct the rule vectors; also
    used to validate assigned lineages
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:7be51495a1e7ba2a4abc774c7a3d55e0e676e93dcdd6d405a350815f0c0b71cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - data.table
  - lineagespot
  - stringr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# lineage-rule-vector-construction

## Summary

Constructs the per-lineage rule vectors used by lineagespot to quantify how well sample mutations satisfy SARS-CoV-2 lineage definitions (e.g., Pangolin rules against reference genome NC_045512): one reference-base position entry per lineage-defining site plus inequality rules requiring observed bases to differ from the reference, with multi-allelic ALT fields split into separate rules.

## When to use

Use when you need to convert a SARS-CoV-2 lineage definition (Pangolin rule or outbreak.info profile) into an evaluable rule vector against the NC_045512 reference, so VCF-derived variants can be scored for rule satisfaction per lineage.

## When NOT to use

- Input is not variant call data (e.g., raw FASTQ reads) — reads must first be mapped to NC_045512 with Minimap2 and variant-called (e.g., freebayes) before rule evaluation
- The goal is whole-genome phylogenetic placement of a single high-coverage consensus genome, where standard Pangolin on a consensus FASTA is the established route rather than rule-satisfaction scoring of low-frequency wastewater variants

## Inputs

- VCF file(s) of detected nucleotide mutations (e.g., from freebayes) with allele frequency
- SARS-CoV-2 reference genome NC_045512 position/base table
- Lineage definitions retrieved as Pangolin rules or outbreak.info lineage-characteristic mutation profiles

## Outputs

- Per-lineage rule-vector table (CSV) with position, reference base, and single-ALT inequality rules
- Per-sample counts of variants satisfying each lineage rule set and the proportion of satisfied lineage rules

## How to apply

Load the VCF mutation calls and the SARS-CoV-2 reference positions (NC_045512) in R using the lineagespot package with data.table for fast tabular reading. For each lineage, build the rule vector by setting each position entry to the reference base and appending additional decision rules requiring the observed base to differ from that reference base. Remove from the first vector any positions whose reference read depth equals zero, and split comma-delimited multi-allelic ALT fields into separate single-ALT rules using stringr. The resulting vector is then used to evaluate each read/variant record and tally rule satisfaction (number of variants to number of lineage rules) for a given allele-frequency threshold.

## Related tools

- **lineagespot** (R framework that builds the lineage rule vectors and scores variants against them) — github.com/BiodataAnalysisGroup/lineagespot
- **data.table** (Fast tabular reading of VCF calls and reference positions in R)
- **stringr** (Splits comma-delimited multi-allelic ALT fields into separate single-ALT rules)
- **Pangolin** (Source of lineage definitions/rules used to construct the rule vectors; also used to validate assigned lineages)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- Output table exists in CSV format and contains per-lineage rule entries with position, reference base, and ALT rules
- Assigned lineages agree with the entries of the Pangolin lineage report
- Positions with zero reference read depth are absent from the first vector
- No comma-delimited multi-allelic entries remain — each rule carries exactly one ALT
- Rule-satisfaction proportion (variants to lineage rules) is computable for each lineage at the chosen allele-frequency threshold

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage (e.g., one group satisfies rule A of lineage L, another satisfies rule B of the same lineage L, and L is incorrectly assigned)
- Lineage detection depends on upstream variant calling quality; freebayes at low mutation frequencies (< 0.01) was chosen, and positions with zero reference depth are excluded
- Mutation identification may be hindered by viral adsorption and decay in wastewater

## Evidence

- [discussion] zero-depth filter: "positions with reference read depth equal to zero are removed from the first vector"
- [results] lineage definition source: "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [discussion] read filter context: "only the paired-end sequences were retained, while any other (unmatched, multiple mappings, etc.) were removed"
- [readme] lineagespot purpose: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [abstract] quantification method: "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
