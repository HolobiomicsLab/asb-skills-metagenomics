---
name: yaml-configuration-file-authoring
description: 'Use when generating a VueGen report: the practitioner has a directory
  of bioinformatics output components (plots, tables, networks, Markdown text, HTML
  components) and must describe their report organization in YAML before invoking
  the vuegen CLI.'
when_to_use_negative:
- The output directory layout already encodes the report structure and you can invoke
  VueGen with --directory instead of hand-writing a YAML config
- The upstream tooling already emits a VueGen-compatible YAML config (e.g., a Nextflow
  process that generates one), making manual authoring redundant
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VueGen
  role: Consumes the YAML configuration to assemble and render reports from the declared
    components; distributed as a Python package, Docker image, nf-core module, and
    desktop application
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Quarto
  role: Rendering engine required for document and presentation report types; availability
    is checked with quarto check
- name: Nextflow/nf-core
  role: Workflow context in which the VueGen nf-core module's process script generates
    the VueGen-compatible YAML from staged upstream component files
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:eb18b21cbdebc0b29712282a3c7357e1d76a9800b3d83659c07571c91fa30cec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Docker
  - Jupyter
  - Nextflow/nf-core
  - Quarto
  - VueGen
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# yaml-configuration-file-authoring

## Summary

Authoring a VueGen-compatible YAML configuration file that declares the structure of a scientific report — sections, subsections, and their components (component_type and file_path entries such as CSV dataframes, PNG/plotly/altair plots, and EDGELIST interactive networks) — so VueGen can automatically render reports in formats like HTML, PDF, or Streamlit.

## When to use

Use when generating a VueGen report: the practitioner has a directory of bioinformatics output components (plots, tables, networks, Markdown text, HTML components) and must describe their report organization in YAML before invoking the vuegen CLI.

## When NOT to use

- The output directory layout already encodes the report structure and you can invoke VueGen with --directory instead of hand-writing a YAML config
- The upstream tooling already emits a VueGen-compatible YAML config (e.g., a Nextflow process that generates one), making manual authoring redundant

## Inputs

- Component files: CSV dataframes (e.g., metadata_random_subset.csv, phyla_counts_subset.csv)
- Static plot files (PNG) and interactive plot JSON (plotly/altair)
- Interactive network files (EDGELIST CSV with plot_type INTERACTIVE_NETWORK)
- Markdown text and HTML component files
- Choice of report type (Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, Jupyter)
- Example data directory (e.g., example_data/Earth_microbiome_vuegen_demo_notebook)

## Outputs

- VueGen-compatible YAML configuration file specifying sections, subsections, and components (component_type/file_path entries)
- Generated report (e.g., earth_microbiome_vuegen_report.html) rendered by VueGen/Quarto

## How to apply

Enumerate the component files in the output directory (e.g., metadata_random_subset.csv, phyla_counts_subset.csv, static PNG plots, plotly/altair JSON plots, and EDGELIST CSV files with plot_type INTERACTIVE_NETWORK) and classify each by component_type (dataframe, plot, markdown, html, apicall, chatbot). Structure the YAML into the report's sections and subsections — for the Earth Microbiome Project case study: Exploratory Data Analysis, Metagenomics (alpha diversity, average copy number, nestedness, Shannon entropy), and Network Analysis — with each component entry pointing to its file_path. Pass the config to the vuegen CLI with a --report-type flag (Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, or Jupyter); alternatively, use --directory and let VueGen work from the folder. Ensure Quarto CLI is available (quarto check, install from quarto.org if missing) for document/presentation report types, and leave streamlit_autorun at its default False. In Nextflow contexts (nf-core module), the process script can auto-generate the VueGen-compatible YAML from staged upstream output files before executing the CLI.

## Related tools

- **VueGen** (Consumes the YAML configuration to assemble and render reports from the declared components; distributed as a Python package, Docker image, nf-core module, and desktop application) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Quarto** (Rendering engine required for document and presentation report types; availability is checked with quarto check)
- **Nextflow/nf-core** (Workflow context in which the VueGen nf-core module's process script generates the VueGen-compatible YAML from staged upstream component files)

## Examples

```
vuegen --config report_config.yaml --report-type html  # or: vuegen --directory example_data/Earth_microbiome_vuegen_demo_notebook --report-type html
```

## Evaluation signals

- The YAML parses cleanly and each entry's component_type matches the actual file format at file_path (CSV dataframe, PNG, plotly/altair JSON, EDGELIST INTERACTIVE_NETWORK)
- Running vuegen with the config completes without errors and Quarto is available for document/presentation report types
- Every configured section/subsection of the report is populated — e.g., the three EMP sections (Exploratory Data Analysis, Metagenomics, Network Analysis) all render
- The rendered report displays tables via aggrid and plots correctly; the output report artifact (e.g., earth_microbiome_vuegen_report.html) exists and is well-formed HTML

## Limitations

- The YAML must match VueGen's supported component_type vocabulary; unrecognized components or mismatched file formats will not render
- Report types such as PDF, DOCX, PPTX require the Quarto CLI to be installed; a missing Quarto installation fails rendering
- Acceptance is largely qualitative — the paper reports no quantitative benchmark for report generation, so correctness is judged by successful execution and visual inspection of sections/components

## Evidence

- [abstract] Directory-based config: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [abstract] Supported report formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [intro] Tool definition: "VueGen is a Python package that automates the creation of scientific reports."
- [full_text] Nextflow module role: [evidence span withheld — anchor status: not_found]
