---
name: pangolin-lineage-assignment-interpretation
description: Use when sample mutations in a VCF must be matched against SARS-CoV-2
  lineage-defining rules (Pangolin or outbreak.info) to explain or validate a lineage
  assignment from wastewater NGS data.
when_to_use_negative:
- Input is raw FASTQ rather than variant calls — variant calling (freebayes against
  NC_045512) must precede rule matching
- A single high-confidence lineage assignment is needed for pure clinical isolates
  without concern for mixed populations — the rule-counting assumption can misassign
  when different reads satisfy different rules of the same lineage
- Lineage definitions are needed for viruses other than SARS-CoV-2 mapped to NC_045512
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: lineagespot
  role: R framework that builds lineage rule vectors and tallies satisfied rules to
    detect SARS-CoV-2 lineages from VCF files
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language of lineagespot; used with data.table for fast tabular
    reading and stringr for splitting multi-allelic ALTs
- name: Pangolin
  role: Source of lineage-defining rules and the lineage report against which rule-satisfaction
    outcomes are compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:5aad6667d56ef0c38fb3d1dc5ace0e6b4cf024c2aa474ada7a09c27189d96916
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pangolin
  - R
  - lineagespot
  - stringr
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pangolin-lineage-assignment-interpretation

## Summary

Reconstructs how lineagespot interprets Pangolin lineage rules by building a per-lineage rule vector (position rules plus four inequality rules per position) over the NC_045512 reference and tallying satisfied rules against sample variants, so lineage assignments can be reproduced and critiqued.

## When to use

Use when sample mutations in a VCF must be matched against SARS-CoV-2 lineage-defining rules (Pangolin or outbreak.info) to explain or validate a lineage assignment from wastewater NGS data.

## When NOT to use

- Input is raw FASTQ rather than variant calls — variant calling (freebayes against NC_045512) must precede rule matching
- A single high-confidence lineage assignment is needed for pure clinical isolates without concern for mixed populations — the rule-counting assumption can misassign when different reads satisfy different rules of the same lineage
- Lineage definitions are needed for viruses other than SARS-CoV-2 mapped to NC_045512

## Inputs

- VCF file(s) of called variants from wastewater NGS data (e.g., freebayes calls against NC_045512)
- SARS-CoV-2 lineage definition rules retrieved as Pangolin rules or outbreak.info profiles
- SARS-CoV-2 reference genome NC_045512 (Wuhan) positions with reference bases and read depths

## Outputs

- CSV table with per-lineage rule entries: position, reference base, and ALT rules
- Per-sample, per-lineage counts of variants and lineage rules and the proportion of satisfied rules
- Lineage assignments comparable against the Pangolin lineage report

## How to apply

Load the VCF mutation calls and the NC_045512 (Wuhan) reference positions in R using the lineagespot package with data.table, and build a rule vector per lineage: set each position entry to the reference base, then append four additional decision rules requiring the observed base to differ from that reference base. Remove positions whose reference read depth equals zero from the first vector, and split comma-delimited multi-allelic ALT fields into separate single-ALT rules with stringr. Evaluate each read/variant record against the lineage rule set and compute, per lineage, the number of variants divided by the number of lineage rules (the proportion of satisfied rules) at a given allele-frequency threshold. Compare rule-satisfaction outcomes against the Pangolin lineage report, noting the caveat that different reads satisfying different rules of the same lineage can still yield that lineage assignment.

## Related tools

- **lineagespot** (R framework that builds lineage rule vectors and tallies satisfied rules to detect SARS-CoV-2 lineages from VCF files) — github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language of lineagespot; used with data.table for fast tabular reading and stringr for splitting multi-allelic ALTs)
- **Pangolin** (Source of lineage-defining rules and the lineage report against which rule-satisfaction outcomes are compared)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- Output table exists in CSV format and contains per-lineage rule entries with position, reference base, and ALT rules
- Assigned lineages agree with the entries in the Pangolin lineage report
- Positions with reference read depth equal to zero are absent from the first rule vector
- Multi-allelic ALT fields are split into separate single-ALT rules (no comma-delimited entries remain)
- Per-lineage proportion of satisfied rules is reported as number of variants to number of lineage rules at the chosen allele-frequency threshold

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage L
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing inconsistencies (primer trimming and remapping omitted for four samples between 19 March 2021 and 15 April 2021) can affect downstream variant and rule evaluation

## Evidence

- [discussion] incorrect-assignment caveat: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
- [discussion] depth-zero filter: "positions with reference read depth equal to zero are removed from the first vector"
- [results] lineage rule sources: "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [readme] tool purpose: "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [abstract] implementation language and availability: "lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools"
