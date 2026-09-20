---
name: multi-allelic-alt-allele-handling
description: Use when VCF variant calls from low-frequency wastewater sequencing (freebayes,
  allele frequency < 0.01) contain comma-separated multi-allelic ALT strings that
  must become one rule per ALT before lineage rule matching.
when_to_use_negative:
- VCF records are already biallelic (no comma-delimited ALT fields), so splitting
  would duplicate single-ALT rules
- Positions whose reference read depth equals zero have not yet been removed — zero-depth
  filtering must precede rule-vector construction
- The goal is de novo lineage discovery rather than matching mutations against known
  Pangolin/outbreak.info lineage rules
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: lineagespot
  role: R framework that builds lineage rule vectors from Pangolin rules and evaluates
    VCF variants (including split ALT alleles) against them
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: stringr
  role: Splits comma-delimited multi-allelic ALT fields into separate single-ALT rules
- name: data.table
  role: Fast tabular reading of VCF mutation calls and reference positions in R
- name: freebayes
  role: Source variant caller whose output contains the multi-allelic ALT fields being
    handled
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:80807e13d98e9fbd193956b0732ccc907d81893ef1e8024345174a3c7d3fd90a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - R
  - data.table
  - freebayes
  - lineagespot
  - stringr
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multi-allelic-alt-allele-handling

## Summary

Splits comma-delimited multi-allelic ALT fields from freebayes VCF calls (SARS-CoV-2, NC_045512 reference) into separate single-ALT rules so each alternate allele can be independently matched against Pangolin lineage rule vectors in lineagespot.

## When to use

Use when VCF variant calls from low-frequency wastewater sequencing (freebayes, allele frequency < 0.01) contain comma-separated multi-allelic ALT strings that must become one rule per ALT before lineage rule matching.

## When NOT to use

- VCF records are already biallelic (no comma-delimited ALT fields), so splitting would duplicate single-ALT rules
- Positions whose reference read depth equals zero have not yet been removed — zero-depth filtering must precede rule-vector construction
- The goal is de novo lineage discovery rather than matching mutations against known Pangolin/outbreak.info lineage rules

## Inputs

- VCF files of variant calls (e.g., freebayes output on ARTIC-pipeline wastewater reads)
- SARS-CoV-2 reference genome NC_045512 positions with reference bases
- Pangolin lineage rule definitions (rule vectors per lineage)

## Outputs

- Expanded per-lineage rule table (CSV) with one entry per position/ALT rule, including position, reference base, and individual ALT rules
- Per-sample tally of satisfied lineage rules (number of variants to number of lineage rules) comparable with Pangolin lineage reports

## How to apply

Load the VCF mutation calls and NC_045512 reference positions in R (lineagespot package, data.table for fast tabular reading). Build the per-lineage rule vector by setting each position entry to the reference base and appending four inequality rules requiring the observed base to differ from the reference base. Before rule matching, remove from the first vector any positions whose reference read depth equals zero, then split comma-delimited multi-allelic ALT fields into separate single-ALT rules using stringr. Rationale: freebayes emits multiple ALT alleles per position, so a single multi-allelic entry would otherwise be evaluated as one lineage rule instead of independent allele rules, distorting the variants-per-lineage-to-rules proportion used for lineage quantification.

## Related tools

- **lineagespot** (R framework that builds lineage rule vectors from Pangolin rules and evaluates VCF variants (including split ALT alleles) against them) — https://github.com/BiodataAnalysisGroup/lineagespot
- **stringr** (Splits comma-delimited multi-allelic ALT fields into separate single-ALT rules)
- **data.table** (Fast tabular reading of VCF mutation calls and reference positions in R)
- **freebayes** (Source variant caller whose output contains the multi-allelic ALT fields being handled)

## Examples

```
rules <- vcf[ref_read_depth > 0, .(POS, REF)]; single_alts <- stringr::str_split(vcf$ALT, ",", simplify = FALSE)
```

## Evaluation signals

- Output table exists in CSV format and contains per-lineage rule entries with position, reference base, and ALT rules
- No row in the expanded rule set retains a comma in its ALT field after stringr splitting
- All positions with reference read depth equal to zero are absent from the first rule vector
- Assigned lineages agree with the Pangolin lineage report entries for the same sample
- Per-lineage counts of variants relative to the number of lineage rules are recomputable from the expanded single-ALT table

## Limitations

- Reads satisfying different rules of the same lineage can still yield that lineage assignment, producing incorrect lineage calls when different read groups satisfy different rules of lineage L
- Mutation identification may be hindered by viral adsorption and decay in wastewater samples
- Preprocessing was inconsistent in the source study: for four samples between 19 March 2021 and 15 April 2021, primer trimming and remapping to the reference genome were not applied

## Evidence

- [discussion] positions with reference read depth equal to zero are removed from the first vector: "positions with reference read depth equal to zero are removed from the first vector"
- [readme] lineagespot identifies SARS-CoV-2 mutations from VCF files: "Lineagespot is a framework written in R, and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [discussion] lineage assignment assumption caveat: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
