---
name: php-web-module-implementation
description: Use when building or modifying a web-based feature (e.g., a sequencing
  submission form or custom field handler) inside MetaLIMS, where PHP 5.5.25, the
  mysqli module, and MySQL 5.5.43 are the deployment stack.
when_to_use_negative:
- The lab needs a LIMS that records NGS or analysis data itself — MetaLIMS intentionally
  omits recording NGS or analysis data
- The task requires active maintenance guarantees — MetaLIMS is no longer being actively
  maintained and has not been updated for recent Ubuntu, MySQL, PHP, or Apache versions
- The goal is a full-featured commercial LIMS with instrument integration and complex
  workflows rather than simple sample metadata storage
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaLIMS
  role: The web application whose PHP codebase hosts the module; free and open source,
    deployed and customized via GitHub
  repo: https://github.com/cheinle/MetaLIMS
- name: PHP 5.5.25
  role: Server-side language used to implement MetaLIMS modules, using the mysqli
    module to query the MySQL database
- name: MySQL 5.5.43
  role: Relational backend storing the sample metadata records that the module queries
    via mysqli
- name: Php Excel
  role: Open-source package used by MetaLIMS to render spreadsheet outputs such as
    the sequencing submission form
- name: XAMPP 3.2.1
  role: Bundle used to build and test MetaLIMS's web-based interface with HTML5
- name: DataTables
  role: Open-source jQuery-based package used by MetaLIMS to display stored records
    in the web interface
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/gix025
    title: MetaLIMS, a simple open-source laboratory information management system
      for small metagenomic labs
  source_content_hash: sha256:58e8d57a1729d394c1d87a95053953d7593a63d83b8b2f1c4b59298a7aeb7c21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cheinle/MetaLIMS
  tools:
  - DataTables
  - MetaLIMS
  - MySQL 5.5.43
  - PHP
  - PHP 5.5.25
  - Php Excel
  - XAMPP 3.2.1
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# php-web-module-implementation

## Summary

Implement and deploy a PHP/MySQL web module inside MetaLIMS, a free open-source LIMS built on XAMPP/Apache/HTML5, so that stored sample metadata can be queried, rendered (e.g., with Php Excel and DataTables), and exported for downstream tasks such as Clarity LIMS submission.

## When to use

Use when building or modifying a web-based feature (e.g., a sequencing submission form or custom field handler) inside MetaLIMS, where PHP 5.5.25, the mysqli module, and MySQL 5.5.43 are the deployment stack.

## When NOT to use

- The lab needs a LIMS that records NGS or analysis data itself — MetaLIMS intentionally omits recording NGS or analysis data
- The task requires active maintenance guarantees — MetaLIMS is no longer being actively maintained and has not been updated for recent Ubuntu, MySQL, PHP, or Apache versions
- The goal is a full-featured commercial LIMS with instrument integration and complex workflows rather than simple sample metadata storage

## Inputs

- Sample metadata records stored in the MetaLIMS MySQL database (generic sample collection fields, DNA/RNA processing fields, lab-specific custom fields)
- MetaLIMS codebase cloned from github.com/cheinle/MetaLIMS
- Target export specification (e.g., Clarity LIMS submission format fields)

## Outputs

- Server-side PHP module integrated into the MetaLIMS web application (HTML5 interface)
- Rendered submission form or data table (Php Excel spreadsheet, DataTables view)
- Exported delimited spreadsheet such as clarity_submission_form.csv

## How to apply

Implement the module as PHP server-side code running on the MetaLIMS stack (built and tested with XAMPP v3.2.1 and HTML5; production deployments use Apache 2.2.15, MySQL 5.5.43 with the mysqli module, and PHP 5.5.25 on a closed internet network). Use the mysqli interface to query stored sample records, including generic sample collection fields, DNA/RNA processing fields, and any lab-specific custom fields the user has added. Reuse the open-source packages MetaLIMS already bundles — Php Excel for spreadsheet rendering (e.g., producing the Clarity LIMS-compatible sequencing submission form) and DataTables with jQuery libraries for table display. Decision points follow the MetaLIMS wiki: pick hosted installation via AWS Lightsail (with lamp.sh, apache_ssl_16.sh, metalims.sh bash scripts) versus full manual installation before adding the module. Validate that the module's output artifact (e.g., clarity_submission_form.csv) exists, is a valid delimited spreadsheet, and contains one row per stored sample record with the expected fields as columns.

## Related tools

- **MetaLIMS** (The web application whose PHP codebase hosts the module; free and open source, deployed and customized via GitHub) — https://github.com/cheinle/MetaLIMS
- **PHP 5.5.25** (Server-side language used to implement MetaLIMS modules, using the mysqli module to query the MySQL database)
- **MySQL 5.5.43** (Relational backend storing the sample metadata records that the module queries via mysqli)
- **Php Excel** (Open-source package used by MetaLIMS to render spreadsheet outputs such as the sequencing submission form)
- **XAMPP 3.2.1** (Bundle used to build and test MetaLIMS's web-based interface with HTML5)
- **DataTables** (Open-source jQuery-based package used by MetaLIMS to display stored records in the web interface)

## Evaluation signals

- The module runs on the documented production stack (Apache 2.2.15, MySQL 5.5.43 with mysqli, PHP 5.5.25) without modification to core MetaLIMS behavior
- Generated output file (e.g., clarity_submission_form.csv) exists and is a valid delimited spreadsheet
- Exported submission form contains one row per stored sample record in the MySQL database
- The Clarity LIMS-compatible submission fields appear as columns in the exported form
- Lab-specific custom fields added by the user are retained in the module's output alongside generic sample collection and DNA/RNA processing fields

## Limitations

- MetaLIMS is no longer being actively maintained and has not been updated for recent Ubuntu, MySQL, PHP, or Apache releases; usage remains at the user's own discretion
- MetaLIMS does not record NGS or analysis data, so modules built on it cannot cover downstream analysis tracking
- The stack is pinned to older versions (XAMPP v3.2.1, PHP 5.5.25, MySQL 5.5.43, Apache 2.2.15); no quantitative evaluation of deployment/maintenance ease is reported

## Evidence

- [results] MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network.: "MetaLIMS has been deployed on a production server using Apache 2.2.15, MySQL 5.5.43 (including mysqli module), and PHP 5.5.25 on a closed internet network."
- [results] MetaLIMS uses many open-source packages such as Php Excel, DataTables, free html and php login templates, jquery libraries, and Creative Commons wallpapers: "MetaLIMS uses many open-source packages such as Php Excel, DataTables, free html and php login templates, jquery libraries, and Creative Commons wallpapers"
- [readme] MetaLIMS is a free and open source web-based application that is available for deployment and customization via GitHub at no cost.: "MetaLIMS is a free and open source web-based application that is available for deployment and customization via GitHub at no cost."
- [abstract] MetaLIMS can also produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities.: "MetaLIMS can also produce a basic sequencing submission form compatible with the proprietary Clarity LIMS system used by some sequencing facilities."
