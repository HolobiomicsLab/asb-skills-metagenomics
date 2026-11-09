---
name: dna-rna-processing-metadata-annotation
description: Use when building or extending a sample metadata table for metagenomic
  lab workflows that must capture DNA/RNA extraction and processing status per sample
  before sequencing, especially when labs need custom fields beyond a generic schema.
when_to_use_negative:
- When the workflow requires recording NGS or downstream analysis data — MetaLIMS
  is explicitly focused on storing sample metadata prior to sequencing and analysis
  pipelines
- When the lab needs instrument integration or complex workflow management typical
  of elaborate LIMS packages
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaLIMS
  role: Web-based LIMS whose schema defines the generic sample collection and DNA/RNA
    processing fields and provides the admin-driven 'User Created Fields' extension
    mechanism
  repo: https://github.com/cheinle/MetaLIMS
- name: pandas
  role: Used to build and serialize the seed and extended sample metadata table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:004e2355599939658a0fb7d5776f5125210a6723064af5c774c33105589753ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - MetaLIMS
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# DNA/RNA processing metadata annotation

## Summary

A schema-construction skill for annotating metagenomic samples with DNA/RNA processing metadata (extraction and processing status) alongside generic sample collection fields, extended by lab-specific custom fields, as implemented in MetaLIMS.

## When to use

Use when building or extending a sample metadata table for metagenomic lab workflows that must capture DNA/RNA extraction and processing status per sample before sequencing, especially when labs need custom fields beyond a generic schema.

## When NOT to use

- When the workflow requires recording NGS or downstream analysis data — MetaLIMS is explicitly focused on storing sample metadata prior to sequencing and analysis pipelines
- When the lab needs instrument integration or complex workflow management typical of elaborate LIMS packages

## Inputs

- Seed CSV of metagenomic samples with sample identifiers and collection details
- MetaLIMS schema conventions from github.com/cheinle/MetaLIMS documentation
- List of lab-specific custom fields (text, dropdown, or numeric) to append

## Outputs

- sample_metadata_records.csv — one structured record per sample with generic sample collection fields, DNA/RNA processing fields, and at least one lab-specific custom field

## How to apply

First, consult the MetaLIMS schema conventions from the project documentation at github.com/cheinle/MetaLIMS to identify the generic sample collection fields and the DNA/RNA processing fields that MetaLIMS records prior to sequencing and analysis pipelines. Build a seed CSV with rows of metagenomic samples and columns covering generic sample collection metadata (e.g. sample identifier, collection details) plus DNA/RNA processing metadata (e.g. extraction and processing status) using pandas. Then apply MetaLIMS' custom-field extension mechanism by appending lab-specific columns beyond the generic schema — MetaLIMS allows admins to add custom text (free text or dropdown) and numeric entries, which can be marked required and appear under the 'User Created Fields' tab. Serialize the extended table, one structured record per sample, to sample_metadata_records.csv.

## Related tools

- **MetaLIMS** (Web-based LIMS whose schema defines the generic sample collection and DNA/RNA processing fields and provides the admin-driven 'User Created Fields' extension mechanism) — https://github.com/cheinle/MetaLIMS
- **pandas** (Used to build and serialize the seed and extended sample metadata table)

## Examples

```
import pandas as pd; df = pd.read_csv('samples_seed.csv'); df['extraction_status'] = 'completed'; df['lab_custom_field'] = 'value'; df.to_csv('sample_metadata_records.csv', index=False)
```

## Evaluation signals

- sample_metadata_records.csv exists at the expected output path
- Every sample row carries values for the generic sample collection fields
- Every sample row carries values for the DNA/RNA processing fields (e.g. extraction and processing status)
- Every sample row includes at least one lab-specific custom field
- Record count in the output file equals the number of seed samples

## Limitations

- MetaLIMS is no longer being actively maintained and has not been updated for changes in recent Ubuntu, MySQL, PHP, and Apache versions; usage remains at the user's own discretion
- MetaLIMS does not record NGS or analysis data, which may be a gap for labs requiring full workflow management
- No quantitative evaluation or user study supports the claim that deployment and maintenance is easy

## Evidence

- [readme] MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines: "MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines."
- [readme] in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the users lab: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the users lab."
- [other] extends via admin-created custom text and numeric entries marked required under 'User Created Fields' tab (task_001 finding): [evidence span withheld — anchor status: not_found]
- [discussion] do not need the extra bulk of recording NGS or analysis data, which is common in many NGS LIMS: "do not need the extra bulk of recording NGS or analysis data, which is common in many NGS LIMS"
- [readme] MetaLIMS is no longer being actively maintained: "MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date."
