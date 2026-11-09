---
name: lims-submission-form-generation
description: Use when a lab needs to submit stored sample metadata (generic collection
  fields, DNA/RNA processing fields, and lab-specific custom fields) to a sequencing
  facility running Clarity LIMS.
when_to_use_negative:
- The lab does not use Clarity LIMS at its sequencing facility and needs a different
  submission template
- The goal is to record or manage NGS or analysis data — MetaLIMS explicitly does
  not store NGS or analysis data
- Sample metadata has not yet been entered into MetaLIMS, so there are no stored records
  to map into the submission form
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaLIMS
  role: Web application that stores the sample metadata and generates/exports the
    Clarity LIMS-compatible sequencing submission form
  repo: https://github.com/cheinle/MetaLIMS
- name: Clarity LIMS
  role: Proprietary target system whose submission format the generated form must
    be compatible with
- name: MySQL 5.5.43
  role: Backend database queried via the mysqli module to retrieve stored sample records
    for the form
- name: PHP 5.5.25
  role: Server-side language used with mysqli to query sample records and render the
    submission form
- name: Php Excel
  role: Open-source package used by MetaLIMS to render the sequencing submission form
    for export
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:7f741b481704aa16025828fc5e058ef338002d8138a1d4277c8da7455e4b9d17
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lims-submission-form-generation

## Summary

Generates a basic sequencing submission form from sample metadata stored in MetaLIMS, formatted for compatibility with the proprietary Clarity LIMS system used by some sequencing facilities.

## When to use

Use when a lab needs to submit stored sample metadata (generic collection fields, DNA/RNA processing fields, and lab-specific custom fields) to a sequencing facility running Clarity LIMS.

## When NOT to use

- The lab does not use Clarity LIMS at its sequencing facility and needs a different submission template
- The goal is to record or manage NGS or analysis data — MetaLIMS explicitly does not store NGS or analysis data
- Sample metadata has not yet been entered into MetaLIMS, so there are no stored records to map into the submission form

## Inputs

- Sample metadata records stored in MetaLIMS (MySQL database via mysqli), including generic sample collection fields, DNA/RNA processing fields, and lab-specific custom fields

## Outputs

- clarity_submission_form.csv — a delimited sequencing submission form compatible with the proprietary Clarity LIMS system, with one row per stored sample

## How to apply

Query the stored sample records from the MySQL database (via the mysqli interface in PHP), including generic sample collection fields, DNA/RNA processing fields, and any lab-specific custom fields the lab has added. Map the retrieved metadata onto the fields required by the Clarity LIMS submission format, retaining lab-specific custom fields as additional columns. Render the mapped records as a basic sequencing submission form using the Php Excel package within the MetaLIMS web application, then export the completed form as clarity_submission_form.csv from the MetaLIMS web interface. Validate that the exported file is a valid delimited spreadsheet containing one row per stored sample with the Clarity LIMS-compatible submission fields present as columns.

## Related tools

- **MetaLIMS** (Web application that stores the sample metadata and generates/exports the Clarity LIMS-compatible sequencing submission form) — https://github.com/cheinle/MetaLIMS
- **Clarity LIMS** (Proprietary target system whose submission format the generated form must be compatible with)
- **MySQL 5.5.43** (Backend database queried via the mysqli module to retrieve stored sample records for the form)
- **PHP 5.5.25** (Server-side language used with mysqli to query sample records and render the submission form)
- **Php Excel** (Open-source package used by MetaLIMS to render the sequencing submission form for export)

## Evaluation signals

- clarity_submission_form.csv exists in the MetaLIMS web interface after export
- The exported file is a valid delimited spreadsheet
- The file contains one row per stored sample record from the MySQL database
- The Clarity LIMS-compatible submission fields are present as columns in the exported file
- Lab-specific custom fields are retained as additional columns rather than dropped during the field mapping

## Limitations

- The form is described as a 'basic' sequencing submission form — compatibility is limited to Clarity LIMS, not other submission systems
- MetaLIMS is no longer being actively maintained and has not been updated for changes in recent versions of Ubuntu, MySQL, PHP, and Apache; usage remains at the user's own discretion
- MetaLIMS does not record NGS or analysis data, so the submission form covers only pre-sequencing sample metadata
- The production deployment described used MySQL 5.5.43 and PHP 5.5.25 on a closed internet network; newer stacks may not be supported

## Evidence

- [abstract] Clarity LIMS compatibility claim: "MetaLIMS can also produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities."
- [abstract] Custom field support: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab."
- [results] Php Excel package usage: "MetaLIMS uses many open-source packages such as Php Excel, DataTables, free html and php login templates, jquery libraries, and Creative Commons wallpapers"
- [results] Production server stack: "MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network."
- [readme] Maintenance status warning: "MetaLIMS is no longer being actively maintained. Current applications mentioned may be out of date."
- [discussion] Scope limitation: "do not need the extra bulk of recording NGS or analysis data, which is common in many NGS LIMS"
