---
name: sample-metadata-schema-modeling
description: Use when building or restructuring a pre-sequencing sample metadata table
  (e.g. CSV of metagenomic samples) that must combine generic collection fields, DNA/RNA
  processing fields, and lab-specific custom fields, as MetaLIMS does.
when_to_use_negative:
- The target system must record NGS or downstream analysis data — MetaLIMS explicitly
  does not store NGS or analysis data, only pre-sequencing metadata
- The input is already a complete, validated metadata table satisfying the generic-plus-custom
  schema
- A full-featured proprietary LIMS workflow (instrument integration, complex workflows)
  is required rather than simple pre-sequencing metadata capture
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaLIMS
  role: Reference open-source web-based LIMS whose schema (generic sample collection
    + DNA/RNA processing fields plus admin-created custom text/dropdown/numeric fields
    under 'User Created Fields') the modeled metadata schema mirrors
  repo: https://github.com/cheinle/MetaLIMS
- name: pandas
  role: Python library used to build the seed CSV of metagenomic samples and append
    lab-specific custom columns when constructing the extended metadata table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:b9227ebd0faa4385684eb5ba6ac3fa8f34007d9e79269a3961c486eb843cf323
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - MetaLIMS
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# sample-metadata-schema-modeling

## Summary

Models a sample metadata storage schema in the style of MetaLIMS: a generic core of sample collection and DNA/RNA processing fields, extensible with admin-created custom text (free text or dropdown) and numeric fields that can be marked required. It lets small metagenomic labs capture lab-specific information prior to sequencing and analysis pipelines.

## When to use

Use when building or restructuring a pre-sequencing sample metadata table (e.g. CSV of metagenomic samples) that must combine generic collection fields, DNA/RNA processing fields, and lab-specific custom fields, as MetaLIMS does.

## When NOT to use

- The target system must record NGS or downstream analysis data — MetaLIMS explicitly does not store NGS or analysis data, only pre-sequencing metadata
- The input is already a complete, validated metadata table satisfying the generic-plus-custom schema
- A full-featured proprietary LIMS workflow (instrument integration, complex workflows) is required rather than simple pre-sequencing metadata capture

## Inputs

- Metagenomic sample list / seed table (e.g. CSV or DataFrame read with pandas)
- MetaLIMS project documentation (github.com/cheinle/MetaLIMS wiki: schema conventions, admin manual)
- Lab-defined custom field specifications (text, dropdown, or numeric; required flags)

## Outputs

- sample_metadata_records.csv — one structured record per sample with generic sample collection fields, DNA/RNA processing fields, and at least one lab-specific custom field

## How to apply

First read the MetaLIMS schema conventions from the project documentation at github.com/cheinle/MetaLIMS to identify the generic sample collection fields and the DNA/RNA processing fields the system records prior to sequencing and analysis pipelines. Build a seed CSV of metagenomic samples with pandas, with columns covering generic sample collection metadata (sample identifier, collection details) and DNA/RNA processing metadata (extraction and processing status). Mirror MetaLIMS' extension mechanism by appending lab-specific columns beyond the generic schema, typed as free text, dropdown, or numeric entries, with required flags where the lab mandates them — these correspond to the 'User Created Fields' tab. Serialize one structured record per sample to sample_metadata_records.csv, keeping each row complete across core and custom fields.

## Related tools

- **MetaLIMS** (Reference open-source web-based LIMS whose schema (generic sample collection + DNA/RNA processing fields plus admin-created custom text/dropdown/numeric fields under 'User Created Fields') the modeled metadata schema mirrors) — https://github.com/cheinle/MetaLIMS
- **pandas** (Python library used to build the seed CSV of metagenomic samples and append lab-specific custom columns when constructing the extended metadata table)

## Examples

```
import pandas as pd; df = pd.read_csv('metagenomic_samples_seed.csv'); df['lab_custom_field'] = ''; df.to_csv('sample_metadata_records.csv', index=False)
```

## Evaluation signals

- sample_metadata_records.csv exists and is readable
- Every sample row carries values for the generic sample collection fields and the DNA/RNA processing fields
- Every sample row includes at least one lab-specific custom field value
- Record count in the output file equals the number of seed samples
- Custom columns conform to MetaLIMS field types (free text, dropdown, or numeric) with required fields populated

## Limitations

- MetaLIMS is no longer being actively maintained and has not been updated for recent Ubuntu, MySQL, PHP, and Apache versions; usage remains at the user's own discretion
- MetaLIMS does not record NGS or analysis data, which may be a gap for labs requiring full workflow management
- No quantitative evaluation or user study supports the claim that deployment and maintenance is easy

## Evidence

- [abstract] in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab"
- [other] MetaLIMS stores generic sample collection and DNA/RNA processing information and extends it via admin-created custom text (free text or dropdown) and numeric entries: [evidence span withheld — anchor status: not_found]
- [readme] The focus of MetaLIMS is to store sample metadata prior to sequencing and analysis pipelines: "MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines."
- [discussion] do not need the extra bulk of recording NGS or analysis data, which is common in many NGS LIMS: "do not need the extra bulk of recording NGS or analysis data, which is common in many NGS LIMS"
- [readme] MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date.: "MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date."
