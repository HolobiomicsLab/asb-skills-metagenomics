---
name: microbiome-diversity-report-component-organization
description: Use when you have raw microbiome analysis outputs (e.g., EMP data, alpha
  diversity, Shannon entropy, network files) that must be assembled into a reproducible
  structured scientific report.
when_to_use_negative:
- You need narrative prose reports with custom layouts beyond what a YAML-configured
  directory of component files can express
- Input files are not among the supported component types (plots, tables, networks,
  Markdown text, HTML components) or lack a parseable plot_type entry
- You cannot install the Quarto CLI dependency, since VueGen's report rendering relies
  on it
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: VueGen
  role: Core tool that reads the YAML configuration, ingests component files, and
    renders the report from EMP data
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Python
  role: Language in which VueGen is distributed as a pip-installable package used
    to run the EMP case study
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Streamlit
  role: Alternative report type (--report-type streamlit) rendering the same EMP components
    as an interactive web application
- name: Quarto
  role: Rendering engine whose availability must be verified (quarto check) before
    VueGen generates the report
- name: VueCore
  role: Python package for creating interactive and static visualizations of multi-omics
    data used as report components
  repo: https://github.com/Multiomics-Analytics-Group/vuecore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:3ed59810900528264c2e1936fe1e4793c76c926340faa4da7269407808ea809e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Python
  - Quarto
  - Streamlit
  - VueCore
  - VueGen
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# microbiome-diversity-report-component-organization

## Summary

Reproduces VueGen's Earth Microbiome Project (EMP) case study: organize microbiome analysis outputs (CSV dataframes, static/interactive plots, network edgelists) in a YAML-configured directory so VueGen can ingest them and render a structured HTML or Streamlit report.

## When to use

Use when you have raw microbiome analysis outputs (e.g., EMP data, alpha diversity, Shannon entropy, network files) that must be assembled into a reproducible structured scientific report.

## When NOT to use

- You need narrative prose reports with custom layouts beyond what a YAML-configured directory of component files can express
- Input files are not among the supported component types (plots, tables, networks, Markdown text, HTML components) or lack a parseable plot_type entry
- You cannot install the Quarto CLI dependency, since VueGen's report rendering relies on it

## Inputs

- EMP example data directory (example_data/Earth_microbiome_vuegen_demo_notebook)
- YAML configuration file specifying report sections and components
- CSV dataframes (e.g., metadata_random_subset.csv, phyla_counts_subset.csv)
- Static plots (PNG)
- Interactive plotly/altair plots (JSON)
- Interactive network EDGELIST CSV with plot_type INTERACTIVE_NETWORK

## Outputs

- Structured HTML report saved as earth_microbiome_vuegen_report.html (or a Streamlit web application)
- Organized output directory containing plots, dataframes, and other components
- Three populated report sections: Exploratory Data Analysis, Metagenomics, Network Analysis

## How to apply

Obtain the EMP example directory (example_data/Earth_microbiome_vuegen_demo_notebook) with its YAML configuration file from the VueGen repository, install VueGen via pip, and verify the Quarto CLI with 'quarto check' (installing from quarto.org if missing). Run 'vuegen --directory ... --report-type html' (or streamlit) so VueGen reads the YAML configuration specifying sections for Exploratory Data Analysis, Metagenomics (alpha diversity, average copy number, nestedness, Shannon entropy), and Network Analysis. VueGen ingests component files — CSV dataframes (metadata_random_subset.csv, phyla_counts_subset.csv), static PNG plots, plotly/altair interactive JSON plots, and an EDGELIST CSV with plot_type INTERACTIVE_NETWORK — and organizes outputs within a directory before rendering. The rationale is that VueGen automates report creation from bioinformatics outputs, letting researchers with minimal coding experience communicate results effectively.

## Related tools

- **VueGen** (Core tool that reads the YAML configuration, ingests component files, and renders the report from EMP data) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Python** (Language in which VueGen is distributed as a pip-installable package used to run the EMP case study) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Streamlit** (Alternative report type (--report-type streamlit) rendering the same EMP components as an interactive web application)
- **Quarto** (Rendering engine whose availability must be verified (quarto check) before VueGen generates the report)
- **VueCore** (Python package for creating interactive and static visualizations of multi-omics data used as report components) — https://github.com/Multiomics-Analytics-Group/vuecore

## Examples

```
vuegen --directory example_data/Earth_microbiome_vuegen_demo_notebook --report-type html
```

## Evaluation signals

- Each of the three YAML-configured sections (Exploratory Data Analysis, Metagenomics, Network Analysis) is populated with its configured components in the rendered report
- CSV dataframes display correctly as tables via aggrid in the report
- Static PNG plots and interactive plotly/altair JSON plots render without errors
- The interactive network built from the EDGELIST CSV (plot_type INTERACTIVE_NETWORK) is displayed correctly
- The final report file earth_microbiome_vuegen_report.html is produced and opens with all sections visible

## Limitations

- Quarto CLI must be installed and available (verified via 'quarto check'); VueGen report rendering fails or is unavailable without it
- Components must follow the expected directory organization and YAML schema (section -> component) for VueGen to ingest them correctly
- Interactive networks must be supplied as EDGELIST CSV with plot_type INTERACTIVE_NETWORK; other network formats are not covered by this workflow

## Evidence

- [results] EMP case study workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Report formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [abstract] Directory-based report generation: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [abstract] VueGen purpose: "We present VueGen, a tool that automates the creation of reports from bioinformatics outputs, allowing researchers with minimal coding experience to communicate their results effectively."
