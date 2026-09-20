---
name: sample-metadata-to-clarity-lims-field-mapping
description: Use when sample metadata is stored in MetaLIMS (MySQL/mysqli backend)
  and a sequencing facility requires a Clarity LIMS-compatible submission form derived
  from that metadata.
when_to_use_negative:
- The target sequencing facility does not use the proprietary Clarity LIMS system
  — the form is specifically Clarity LIMS-compatible
- NGS or analysis data need to be recorded or exported — MetaLIMS intentionally does
  not record NGS or analysis data
- No sample metadata has yet been entered into MetaLIMS (empty MySQL sample tables)
  — there are no records to map into the submission form
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3071
tools:
- name: MetaLIMS
  role: Web application that stores sample metadata and produces the Clarity LIMS-compatible
    sequencing submission form
  repo: https://github.com/cheinle/MetaLIMS
- name: Clarity LIMS
  role: Proprietary sequencing facility system that the generated submission form
    must be compatible with
- name: MySQL 5.5.43
  role: Database (with mysqli module) from which stored sample metadata is queried
    for the form
- name: PHP 5.5.25
  role: Server-side language using mysqli to retrieve and map sample metadata fields
- name: Php Excel
  role: Open-source package used to render the sequencing submission form spreadsheet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:ee789a2f3997bac1a032ad79ad4bdf1ccd9532d513cc1669cf5115ad4fe1eb19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - Clarity LIMS
  - MetaLIMS
  - MySQL 5.5.43
  - PHP
  - PHP 5.5.25
  - Php Excel
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sample-metadata-to-clarity-lims-field-mapping

## Summary

Generates a basic sequencing submission form compatible with the proprietary Clarity LIMS system by mapping stored MetaLIMS sample metadata (generic sample collection, DNA/RNA processing, and lab-specific custom fields) from MySQL into a spreadsheet exported via the Php Excel package.

## When to use

Use when sample metadata is stored in MetaLIMS (MySQL/mysqli backend) and a sequencing facility requires a Clarity LIMS-compatible submission form derived from that metadata.

## When NOT to use

- The target sequencing facility does not use the proprietary Clarity LIMS system — the form is specifically Clarity LIMS-compatible
- NGS or analysis data need to be recorded or exported — MetaLIMS intentionally does not record NGS or analysis data
- No sample metadata has yet been entered into MetaLIMS (empty MySQL sample tables) — there are no records to map into the submission form

## Inputs

- Sample metadata records stored in MetaLIMS MySQL database (generic sample collection fields, DNA/RNA processing fields, lab-specific custom fields)
- MetaLIMS web application deployed on Apache/MySQL/mysqli/PHP stack

## Outputs

- clarity_submission_form.csv — a delimited spreadsheet with one row per stored sample record and Clarity LIMS-compatible submission fields as columns

## How to apply

Query the stored sample records — generic sample collection fields, DNA/RNA processing fields, and any lab-specific custom fields — from the MySQL database using the mysqli interface in PHP. Map the retrieved metadata fields to the fields required by the Clarity LIMS submission format, retaining lab-specific custom fields as additional columns. Render the mapped records as a basic sequencing submission form using the Php Excel package within the MetaLIMS web application. Finally, export the completed submission form from the MetaLIMS web interface; validation is confirming the exported file exists, parses as a delimited spreadsheet, and contains one row per stored sample record with Clarity LIMS-compatible submission fields present as columns. Rationale: MetaLIMS stores metadata prior to sequencing and analysis pipelines, so this export bridges lab-internal records and the proprietary system used by sequencing facilities.

## Related tools

- **MetaLIMS** (Web application that stores sample metadata and produces the Clarity LIMS-compatible sequencing submission form) — https://github.com/cheinle/MetaLIMS
- **Clarity LIMS** (Proprietary sequencing facility system that the generated submission form must be compatible with)
- **MySQL 5.5.43** (Database (with mysqli module) from which stored sample metadata is queried for the form)
- **PHP 5.5.25** (Server-side language using mysqli to retrieve and map sample metadata fields)
- **Php Excel** (Open-source package used to render the sequencing submission form spreadsheet)

## Evaluation signals

- clarity_submission_form.csv exists in the MetaLIMS web interface output
- The exported file is a valid delimited spreadsheet (parseable as CSV)
- The file contains exactly one row per stored sample record in the MySQL database
- Clarity LIMS-compatible submission fields are present as columns in the exported file
- Lab-specific custom fields defined by the user's lab are retained as additional columns

## Limitations

- MetaLIMS is no longer being actively maintained and has not been updated for changes in recent versions of Ubuntu, MySQL, PHP, and Apache; usage remains at the user's own discretion
- The submission form is only 'basic' — compatible with the proprietary Clarity LIMS used by some sequencing facilities, not all LIMS systems
- MetaLIMS does not record NGS or analysis data, so the submission form covers pre-sequencing sample metadata only

## Evidence

- [abstract] Clarity LIMS form capability: "MetaLIMS can also produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities."
- [abstract] custom fields: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab."
- [results] Php Excel usage: "MetaLIMS uses many open-source packages such as Php Excel, DataTables, free html and php login templates, jquery libraries, and Creative Commons wallpapers"
- [results] production stack: "MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network."
- [readme] no active maintenance: "MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date."
