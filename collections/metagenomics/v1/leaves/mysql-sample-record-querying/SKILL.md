---
name: mysql-sample-record-querying
description: Use when sample metadata held in the MetaLIMS MySQL database must be
  queried through PHP's mysqli interface and mapped — including lab-specific custom
  fields — to build a Clarity LIMS-compatible sequencing submission form.
when_to_use_negative:
- The lab does not need to interface with the proprietary Clarity LIMS system used
  by some sequencing facilities — the mapping is Clarity-specific
- The sample metadata is not stored in MetaLIMS's MySQL schema (e.g. raw spreadsheets
  or another LIMS's database) and cannot be queried via mysqli
- The requirement is to record or manage NGS or analysis data, which MetaLIMS explicitly
  does not handle
edam_operation: http://edamontology.org/operation_2422
tools:
- name: MetaLIMS
  role: Web application whose MySQL schema holds the sample metadata and which renders
    the Clarity LIMS submission form
  repo: https://github.com/cheinle/MetaLIMS
- name: MySQL 5.5.43
  role: Production database queried for sample records through its mysqli module
- name: PHP 5.5.25
  role: Server-side language executing the mysqli sample-record queries in the MetaLIMS
    web app
- name: Php Excel
  role: Open-source package used to render the queried records as the sequencing submission
    form spreadsheet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:61ce08fad93fe26abcdf640b2a44c5750827de345e54201b7f08ff52af15877d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - MetaLIMS
  - MySQL
  - MySQL 5.5.43
  - PHP
  - PHP 5.5.25
  - Php Excel
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mysql-sample-record-querying

## Summary

Retrieves stored sample metadata (generic sample collection fields, DNA/RNA processing fields, and lab-specific custom fields) from MetaLIMS's MySQL 5.5.43 database via the PHP mysqli module, as the data-extraction step that feeds Clarity LIMS-compatible sequencing submission form generation.

## When to use

Use when sample metadata held in the MetaLIMS MySQL database must be queried through PHP's mysqli interface and mapped — including lab-specific custom fields — to build a Clarity LIMS-compatible sequencing submission form.

## When NOT to use

- The lab does not need to interface with the proprietary Clarity LIMS system used by some sequencing facilities — the mapping is Clarity-specific
- The sample metadata is not stored in MetaLIMS's MySQL schema (e.g. raw spreadsheets or another LIMS's database) and cannot be queried via mysqli
- The requirement is to record or manage NGS or analysis data, which MetaLIMS explicitly does not handle

## Inputs

- MetaLIMS MySQL 5.5.43 database with stored sample metadata records (accessed via the mysqli module)
- Generic sample collection fields and DNA/RNA processing fields per sample
- Lab-specific custom fields added by the user in MetaLIMS
- User selection of sample records to include in the submission

## Outputs

- Retrieved sample metadata rows (one per stored sample) via the PHP mysqli interface
- Mapped records rendered as a basic sequencing submission form using the Php Excel package
- clarity_submission_form.csv exported from the MetaLIMS web interface, with Clarity LIMS-compatible submission fields as columns

## How to apply

Run SELECT queries against the MetaLIMS MySQL 5.5.43 database using PHP 5.5.25's mysqli module (as deployed on the production Apache 2.2.15 server), pulling generic sample collection fields, DNA/RNA processing fields, and any lab-defined custom fields for the selected sample records. Map the retrieved metadata columns onto the fields required by the proprietary Clarity LIMS submission format, keeping lab-specific custom fields as additional columns rather than dropping them. Hand the mapped records to the Php Excel package inside the MetaLIMS web application, which renders the basic sequencing submission form and exports it as clarity_submission_form.csv from the web interface. Rationale: MetaLIMS's core purpose is storing sample metadata prior to sequencing, so this query step is the bridge between stored records and downstream submission to facilities running Clarity LIMS.

## Related tools

- **MetaLIMS** (Web application whose MySQL schema holds the sample metadata and which renders the Clarity LIMS submission form) — https://github.com/cheinle/MetaLIMS
- **MySQL 5.5.43** (Production database queried for sample records through its mysqli module)
- **PHP 5.5.25** (Server-side language executing the mysqli sample-record queries in the MetaLIMS web app)
- **Php Excel** (Open-source package used to render the queried records as the sequencing submission form spreadsheet)

## Examples

```
$res = mysqli_query($db, 'SELECT * FROM samples');
```

## Evaluation signals

- clarity_submission_form.csv exists after export from the MetaLIMS web interface
- The exported file is a valid delimited spreadsheet
- The file contains one row per stored sample record that was queried
- The Clarity LIMS-compatible submission fields are present as columns
- Lab-specific custom fields queried from the database are retained as additional columns rather than discarded

## Limitations

- MetaLIMS is no longer actively maintained, so deployments on recent MySQL, PHP, Apache, or Ubuntu versions are untested and usage is at the user's own discretion
- MetaLIMS does not record NGS or analysis data, so this querying skill covers only pre-sequencing sample metadata
- MetaLIMS was built and tested with specific versions (Apache 2.2.15, MySQL 5.5.43, PHP 5.5.25) on a closed internet network; other environments were not validated

## Evidence

- [results] production deployment stack: "MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network."
- [abstract] field scope of stored metadata: "in addition to storing generic sample collection information and DNA/RNA processing information, the user can also add fields specific to the user's lab."
- [abstract] Clarity LIMS form purpose: "MetaLIMS can also produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities."
- [results] Php Excel package used for form rendering: "MetaLIMS uses many open-source packages such as Php Excel, DataTables, free html and php login templates, jquery libraries"
- [readme] purpose of stored metadata: "MetaLIMS focus is to store sample collection and processing metadata prior to sequencing and analysis pipelines."
