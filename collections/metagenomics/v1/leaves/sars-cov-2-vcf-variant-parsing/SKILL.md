---
name: sars-cov-2-vcf-variant-parsing
description: Use when input is one or more VCF variant-calling files from SARS-CoV-2
  wastewater or clinical NGS data and the goal is to evaluate detected mutations against
  Pangolin/outbreak.info lineage-defining rules.
when_to_use_negative:
- Input is raw FASTQ reads rather than called variants — variant calling (e.g., freebayes
  on reads mapped to NC_045512 with Minimap2) must happen first
- The goal is de novo mutation discovery or phylogenetic placement rather than scoring
  samples against known lineage-defining mutation rules
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0781
tools:
- name: lineagespot
  role: R framework that reads VCF files and scores variants against SARS-CoV-2 lineage-defining
    mutation rules
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: data.table
  role: Fast tabular reading of VCF-derived tables and reference positions in R
- name: stringr
  role: Splitting comma-delimited multi-allelic ALT fields into separate single-ALT
    rules
- name: Pangolin
  role: Source of lineage definitions used to validate rule-satisfaction results
- name: freebayes
  role: Variant caller producing the low-frequency (< 0.01) VCF inputs consumed by
    this parsing step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:a9b88ea67fc88a1c6a8d1ae870577314b45c4b016037b762feb595aab1bdba57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - data.table
  - freebayes
  - lineagespot
  - stringr
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# sars-cov-2-vcf-variant-parsing

## Summary

Parse VCF mutation calls against the SARS-CoV-2 reference genome NC_045512 with the lineagespot R package (using data.table and stringr) to build lineage rule vectors — one position rule per reference base plus four inequality rules — and compute how many lineage rules each sample's variants satisfy.

## When to use

Use when input is one or more VCF variant-calling files from SARS-CoV-2 wastewater or clinical NGS data and the goal is to evaluate detected mutations against Pangolin/outbreak.info lineage-defining rules.

## When NOT to use

- Input is raw FASTQ reads rather than called variants — variant calling (e.g., freebayes on reads mapped to NC_045512 with Minimap2) must happen first
- The goal is de novo mutation discovery or phylogenetic placement rather than scoring samples against known lineage-defining mutation rules

## Inputs

- VCF file(s) of SARS-CoV-2 variant calls (e.g., from freebayes), potentially with multi-allelic comma-delimited ALT fields
- SARS-CoV-2 reference genome positions (NC_045512)
- Lineage-defining rules (Pangolin rules or outbreak.info mutation profiles)
- Allele-frequency threshold parameter

## Outputs

- CSV table with per-lineage rule entries containing position, reference base, and ALT rules
- Per-sample count of variants and proportion of satisfied lineage rules per lineage
- Lineage assignments comparable against a Pangolin lineage report

## How to apply

Load the VCF mutation calls and the NC_045512 reference genome positions in R with the lineagespot package, using data.table for fast tabular reading. Build a rule vector per lineage: set each position entry to the reference base and append four additional decision rules requiring the observed base to differ from that reference base. Remove positions whose reference read depth equals zero from the first vector, and split comma-delimited multi-allelic ALT fields into separate single-ALT rules with stringr. Evaluate each read/variant record against the lineage rule set and tally rule satisfaction as the proportion of satisfied lineage rules (number of variants to number of lineage rules) per sample at the chosen allele-frequency threshold.

## Related tools

- **lineagespot** (R framework that reads VCF files and scores variants against SARS-CoV-2 lineage-defining mutation rules) — github.com/BiodataAnalysisGroup/lineagespot
- **data.table** (Fast tabular reading of VCF-derived tables and reference positions in R)
- **stringr** (Splitting comma-delimited multi-allelic ALT fields into separate single-ALT rules)
- **Pangolin** (Source of lineage definitions used to validate rule-satisfaction results)
- **freebayes** (Variant caller producing the low-frequency (< 0.01) VCF inputs consumed by this parsing step)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); results <- lineagespot(vcf_fls = list("sample.vcf"), voc = "B.1.1.7", af_threshold = 0.01)
```

## Evaluation signals

- Output table exists in CSV format and contains per-lineage rule entries with position, reference base, and ALT rules
- Positions with reference read depth equal to zero are absent from the first rule vector
- No comma-delimited multi-allelic ALT entries remain after stringr splitting
- Assigned lineages agree with the Pangolin lineage report entries for the same samples
- Per-lineage rule-satisfaction proportion (variants / number of lineage rules) is finite and within [0, 1]

## Limitations

- Pangolin-based lineage assignment can be incorrect when different groups of reads satisfy different rules of the same lineage (rule A by one group, rule B by another still yields lineage L)
- Positions with zero reference read depth must be removed or the rule vector is malformed
- Mutation identification may be hindered by viral adsorption and decay in wastewater samples
- Preprocessing inconsistency: primer trimming and remapping were omitted for four samples (19 March – 15 April 2021), which can affect variant calls fed into this step

## Evidence

- [results] lineagespot evaluates lineage rules against sample mutations by computing, for a given allele-frequency threshold, the number of variants per sample for each lineage and the resulting proportion of satisfied lineage rules: [evidence span withheld — anchor status: not_found]
- [methods] Build the rule vector per lineage: set each position entry to the reference base and append four additional decision rules requiring the observed base to differ from that reference base: [evidence span withheld — anchor status: not_found]
- [methods] Remove from the first vector any positions whose reference read depth equals zero, and split comma-delimited multi-allelic ALT fields into separate single-ALT rules using stringr: [evidence span withheld — anchor status: not_found]
- [readme] Lineagespot is a framework written in R, and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s): "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [discussion] Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned.: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
