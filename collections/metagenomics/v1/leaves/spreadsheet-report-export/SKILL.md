---
name: spreadsheet-report-export
description: Use when a lab needs to convert sample records stored in MetaLIMS (generic
  collection fields, DNA/RNA processing fields, and lab-specific custom fields) into
  a submission form for a sequencing facility running Clarity LIMS.
when_to_use_negative:
- The target facility does not use Clarity LIMS — the export produces a form compatible
  specifically with Clarity LIMS, not arbitrary submission templates
- The lab needs to record or export NGS or analysis data — MetaLIMS does not record
  NGS or analysis data, which is 'common in many NGS LIMS'
- The user needs post-sequencing pipeline results — MetaLIMS' focus is storing sample
  metadata prior to sequencing and analysis pipelines
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaLIMS
  role: Web application that stores sample metadata, maps fields to the Clarity LIMS
    submission format, and exposes the export via its web interface
  repo: https://github.com/cheinle/MetaLIMS
- name: Php Excel
  role: Open-source package used by MetaLIMS to render the sequencing submission form
    as a spreadsheet
- name: MySQL 5.5.43 (mysqli)
  role: Database from which stored sample records are queried via the mysqli module
    in PHP
- name: Clarity LIMS
  role: Proprietary target system whose submission format defines the required columns
    of the exported form
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:667dfd885219fc869333281975743961b57e2188491e110115fd8160acb98748
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - Clarity LIMS
  - MetaLIMS
  - MySQL 5.5.43 (mysqli)
  - PHP
  - Php Excel
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spreadsheet-report-export

## Summary

Export stored sample metadata from MetaLIMS as a basic sequencing submission form (clarity_submission_form.csv) compatible with the proprietary Clarity LIMS system used by some sequencing facilities.

## When to use

Use when a lab needs to convert sample records stored in MetaLIMS (generic collection fields, DNA/RNA processing fields, and lab-specific custom fields) into a submission form for a sequencing facility running Clarity LIMS.

## When NOT to use

- The target facility does not use Clarity LIMS — the export produces a form compatible specifically with Clarity LIMS, not arbitrary submission templates
- The lab needs to record or export NGS or analysis data — MetaLIMS does not record NGS or analysis data, which is 'common in many NGS LIMS'
- The user needs post-sequencing pipeline results — MetaLIMS' focus is storing sample metadata prior to sequencing and analysis pipelines

## Inputs

- Sample records stored in MetaLIMS' MySQL database (generic sample collection fields, DNA/RNA processing fields, user-defined lab-specific fields)
- Clarity LIMS submission format field requirements

## Outputs

- clarity_submission_form.csv, a delimited spreadsheet with one row per stored sample and Clarity LIMS-compatible submission fields as columns
- In-application spreadsheet rendering of the submission form via the Php Excel package

## How to apply

Query the stored sample records from the MySQL database using the mysqli interface in PHP (the production stack is Apache 2.2.15 with MySQL 5.5.43 including mysqli and PHP 5.5.25). Map the retrieved metadata fields to the fields required by the Clarity LIMS submission format, retaining lab-specific custom fields as additional columns so they are not dropped. Render the mapped records as a basic sequencing submission form using the Php Excel package within the MetaLIMS web application, then export it as clarity_submission_form.csv from the MetaLIMS web interface. The rationale is that MetaLIMS deliberately stores only pre-sequencing metadata, so the export is a flat, one-row-per-sample spreadsheet rather than a full NGS workflow record.

## Related tools

- **MetaLIMS** (Web application that stores sample metadata, maps fields to the Clarity LIMS submission format, and exposes the export via its web interface) — https://github.com/cheinle/MetaLIMS
- **Php Excel** (Open-source package used by MetaLIMS to render the sequencing submission form as a spreadsheet)
- **MySQL 5.5.43 (mysqli)** (Database from which stored sample records are queried via the mysqli module in PHP)
- **Clarity LIMS** (Proprietary target system whose submission format defines the required columns of the exported form)

## Evaluation signals

- clarity_submission_form.csv exists after export from the MetaLIMS web interface
- The file is a valid delimited spreadsheet that parses without errors
- The file contains exactly one row per stored sample record in the MySQL database
- The Clarity LIMS-compatible submission fields are present as columns
- Lab-specific custom fields are retained as additional columns rather than being silently dropped

## Limitations

- The submission form is explicitly 'basic' — it covers pre-sequencing metadata, not full sequencing or analysis workflows
- MetaLIMS is no longer actively maintained and has not been updated for recent Ubuntu, MySQL, PHP, or Apache versions; usage is at the user's own discretion
- The tested production stack is dated (Apache 2.2.15, MySQL 5.5.43, PHP 5.5.25 on a closed internet network), so field mapping may need adjustment on modern stacks
- No quantitative evaluation or user study is reported to support claims about deployment and maintenance ease

## Evidence

- [readme] MetaLIMS can produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities.: "MetaLIMS can produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities."
- [results] Php Excel used for spreadsheet rendering: "MetaLIMS uses many open-source packages such as Php Excel, DataTables, free html and php login templates, jquery libraries, and Creative Commons wallpapers"
- [results] Production stack including mysqli: "MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network."
- [abstract] Scope limited to pre-sequencing metadata: "The focus of MetaLIMS is to store sample metadata prior to sequencing and analysis pipelines."
- [abstract] User can add lab-specific fields: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab."
