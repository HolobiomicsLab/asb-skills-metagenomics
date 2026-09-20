---
name: tabular-record-serialization
description: Use when sample metadata for multiple metagenomic samples must be captured
  as structured one-row-per-sample records, including lab-specific custom text/dropdown
  or numeric fields beyond a generic collection and processing schema.
when_to_use_negative:
- The workflow requires capturing post-sequencing data — MetaLIMS' focus is storing
  metadata 'prior to sequencing and analysis pipelines' and does not record NGS or
  analysis data
- Records are not per-sample tabular metadata but raw sequencing files or submission
  forms (e.g. the Clarity LIMS sequencing submission form, which is a separate MetaLIMS
  output)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3077
tools:
- name: MetaLIMS
  role: 'Reference schema and application: defines the generic sample collection fields,
    DNA/RNA processing fields, and the admin-created custom field mechanism (text/dropdown
    or numeric, optionally required) that the serialized records must mirror'
  repo: https://github.com/cheinle/MetaLIMS
- name: pandas
  role: Python library used to build the seed sample table, append custom-field columns,
    and write the serialized records to CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:35baad9eea9a454cc384ff5e11c0d4787b472634dc978f44bcd4a6c5a67b4881
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
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tabular-record-serialization

## Summary

Serialize structured sample metadata records — generic sample collection fields, DNA/RNA processing fields, and lab-specific user-created fields — into one-row-per-sample tabular output (CSV), mirroring MetaLIMS' schema in which each sample carries generic plus custom metadata prior to sequencing and analysis pipelines.

## When to use

Use when sample metadata for multiple metagenomic samples must be captured as structured one-row-per-sample records, including lab-specific custom text/dropdown or numeric fields beyond a generic collection and processing schema.

## When NOT to use

- The workflow requires capturing post-sequencing data — MetaLIMS' focus is storing metadata 'prior to sequencing and analysis pipelines' and does not record NGS or analysis data
- Records are not per-sample tabular metadata but raw sequencing files or submission forms (e.g. the Clarity LIMS sequencing submission form, which is a separate MetaLIMS output)

## Inputs

- Seed sample table (e.g. CSV or DataFrame) with one row per metagenomic sample
- List of generic sample collection field names (e.g. sample identifier, collection details)
- List of DNA/RNA processing field names (e.g. extraction and processing status)
- Lab-specific custom field definitions (text/dropdown or numeric, optionally required)

## Outputs

- sample_metadata_records.csv — one structured record per sample with generic collection fields, DNA/RNA processing fields, and at least one user-created custom field per row

## How to apply

First fix the base schema: MetaLIMS stores generic sample collection information and DNA/RNA processing information as its default fields, so every record must carry both. Then extend the schema with lab-specific fields, mirroring MetaLIMS' admin-created custom entries which can be free text, dropdown, or numeric and can be marked required — in a flat table these become additional columns under the equivalent of the 'User Created Fields' tab. Build the seed table from sample rows (e.g. with pandas in Python), append the custom columns, and serialize to a delimited file such as sample_metadata_records.csv with one structured record per sample. Validate that the record count equals the number of seed samples, that generic collection and DNA/RNA processing fields are populated, and that each row includes at least one custom field value.

## Related tools

- **MetaLIMS** (Reference schema and application: defines the generic sample collection fields, DNA/RNA processing fields, and the admin-created custom field mechanism (text/dropdown or numeric, optionally required) that the serialized records must mirror) — https://github.com/cheinle/MetaLIMS
- **pandas** (Python library used to build the seed sample table, append custom-field columns, and write the serialized records to CSV)

## Examples

```
import pandas as pd; df = pd.read_csv('seed_samples.csv'); df.to_csv('sample_metadata_records.csv', index=False)
```

## Evaluation signals

- Output file sample_metadata_records.csv exists and is parseable as a delimited table
- Record count in the output equals the number of seed samples (one structured record per sample)
- Every row carries values for all generic sample collection fields
- Every row carries values for all DNA/RNA processing fields
- Every row contains a value for at least one lab-specific custom field

## Limitations

- MetaLIMS is no longer actively maintained; recent Ubuntu/MySQL/PHP/Apache versions may not be supported and usage is at the user's own discretion
- MetaLIMS does not record NGS or analysis data, so serialized records cover only pre-sequencing sample metadata
- No quantitative evaluation or user study supports claims about deployment ease; schema fidelity relies on the project documentation and wiki

## Evidence

- [readme] MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines.: "MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines."
- [abstract] custom fields beyond generic schema: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab."
- [results] production stack: "MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network."
- [readme] maintenance notice: "MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date."
