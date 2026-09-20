---
name: lims-custom-field-extension
description: Use when a lab needs to record sample information beyond default generic
  sample collection and DNA/RNA processing fields, e.g. building a seed CSV of metagenomic
  samples that must carry lab-specific metadata columns in a MetaLIMS-compatible schema.
when_to_use_negative:
- The lab needs to record NGS or analysis data — MetaLIMS deliberately excludes this
  and targets only pre-sequencing sample collection and DNA/RNA processing metadata
- The lab requires full workflow/instrument-integration LIMS features rather than
  simple sample-metadata storage with custom fields
- An actively maintained LIMS is required — the MetaLIMS README states the project
  is no longer actively maintained
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaLIMS
  role: Open-source web-based LIMS whose custom-field mechanism (admin-created text
    or numeric entries, markable as required, shown under the 'User Created Fields'
    tab) this skill emulates
  repo: https://github.com/cheinle/MetaLIMS
- name: Clarity LIMS
  role: Proprietary sequencing-facility system for which the extended sample records
    can feed a basic sequencing submission form
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:f7a6db6f3d03c190555fc8cc7c6d14e643b23c1caa747546dee427ef64de647b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - Clarity LIMS
  - MetaLIMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lims-custom-field-extension

## Summary

Extend a base LIMS sample-metadata schema (generic sample collection plus DNA/RNA processing fields) with lab-specific custom fields — free text, dropdown, or numeric entries, optionally marked required — mirroring MetaLIMS' 'User Created Fields' mechanism for small metagenomic labs.

## When to use

Use when a lab needs to record sample information beyond default generic sample collection and DNA/RNA processing fields, e.g. building a seed CSV of metagenomic samples that must carry lab-specific metadata columns in a MetaLIMS-compatible schema.

## When NOT to use

- The lab needs to record NGS or analysis data — MetaLIMS deliberately excludes this and targets only pre-sequencing sample collection and DNA/RNA processing metadata
- The lab requires full workflow/instrument-integration LIMS features rather than simple sample-metadata storage with custom fields
- An actively maintained LIMS is required — the MetaLIMS README states the project is no longer actively maintained

## Inputs

- Seed CSV of metagenomic samples (pandas DataFrame) to be populated with metadata
- MetaLIMS schema conventions (generic sample collection fields and DNA/RNA processing fields) from the project documentation/wiki

## Outputs

- sample_metadata_records.csv — one structured record per sample, containing generic sample collection fields, DNA/RNA processing fields, and at least one lab-specific custom field

## How to apply

First read the MetaLIMS schema conventions (github.com/cheinle/MetaLIMS wiki) to identify the generic sample collection fields and the DNA/RNA processing fields that MetaLIMS stores prior to sequencing and analysis pipelines. Build a seed CSV with pandas whose rows are metagenomic samples and whose columns cover those two generic field groups (e.g. sample identifier, collection details, extraction and processing status). Then apply the extension mechanism: append lab-specific columns beyond the generic schema, defining each custom field as free text, a dropdown, or a numeric entry and marking lab-critical ones as required, as an admin would under MetaLIMS' 'User Created Fields' tab. Finally serialize one structured record per sample to sample_metadata_records.csv so the extended table is ready for loading into MetaLIMS or for producing a Clarity LIMS-compatible sequencing submission form.

## Related tools

- **MetaLIMS** (Open-source web-based LIMS whose custom-field mechanism (admin-created text or numeric entries, markable as required, shown under the 'User Created Fields' tab) this skill emulates) — https://github.com/cheinle/MetaLIMS
- **Clarity LIMS** (Proprietary sequencing-facility system for which the extended sample records can feed a basic sequencing submission form)

## Examples

```
import pandas as pd; df = pd.read_csv('samples_seed.csv'); df['lab_ph'] = 7.2; df.to_csv('sample_metadata_records.csv', index=False)
```

## Evaluation signals

- sample_metadata_records.csv exists and is non-empty after serialization
- Every sample row carries values for all generic sample collection fields
- Every sample row carries values for the DNA/RNA processing fields
- Every sample row includes at least one lab-specific custom field beyond the generic schema
- Record count in the output file equals the number of seed samples (no rows added or dropped)

## Limitations

- MetaLIMS is no longer being actively maintained and has not been updated for recent Ubuntu, MySQL, PHP, and Apache versions; usage is at the user's own discretion
- Custom fields are limited to text (free text or dropdown) and numeric entries created by an admin; other field types or per-user schemas are not supported
- MetaLIMS was built and tested on XAMPP v. 3.2.1/HTML5 and deployed with Apache 2.2.15, MySQL 5.5.43, and PHP 5.5.25 on a closed network — newer stacks may not be covered

## Evidence

- [abstract] Custom fields extend the generic schema: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab."
- [results] Custom field types: [evidence span withheld — anchor status: not_found]
- [readme] Scope excludes NGS/analysis data: "MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines."
- [readme] No longer maintained: "MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date."
