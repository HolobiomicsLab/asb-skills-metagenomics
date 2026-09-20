---
name: multi-format-document-rendering
description: Use when a configuration-driven report must be emitted in multiple static
  document/presentation formats from one source file, e.g. converting a report.yaml
  + component directory into report.qmd and rendering it with Quarto.
when_to_use_negative:
- The target is only an interactive Streamlit web application, which VueGen handles
  as a separate report type rather than a Quarto-rendered static document
- The report content is not described by a VueGen YAML configuration (no sections/components
  hierarchy), so there is no config from which to structure the qmd
- Component files cannot be validated against their declared component_type or file_format
  (e.g. a plot_type mismatch), producing an unreliable qmd
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: VueGen
  role: Automates creation of the reports from a YAML config; generates the qmd directory
    structure before rendering
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Quarto
  role: Renders the generated report.qmd into the requested static format (e.g. HTML,
    PDF, PPTX)
- name: quarto-cli
  role: Python library bundled in the pip installation used to invoke Quarto rendering
    programmatically
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Python
  role: Package language used to parse the YAML configuration and orchestrate qmd
    generation
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:2ac873a9d20131a01948a9f75391021e4bb808353559f0f491ebb44d8760bfda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Jupyter
  - Python
  - Quarto
  - VueGen
  - quarto-cli
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-format-document-rendering

## Summary

Render a single Quarto Markdown (qmd) source file — generated from a YAML configuration of report metadata, sections, and components — into any of VueGen's static report formats (PDF, HTML, DOCX, ODT, PPTX, Reveal.js), plus Jupyter notebooks and Streamlit apps.

## When to use

Use when a configuration-driven report must be emitted in multiple static document/presentation formats from one source file, e.g. converting a report.yaml + component directory into report.qmd and rendering it with Quarto.

## When NOT to use

- The target is only an interactive Streamlit web application, which VueGen handles as a separate report type rather than a Quarto-rendered static document
- The report content is not described by a VueGen YAML configuration (no sections/components hierarchy), so there is no config from which to structure the qmd
- Component files cannot be validated against their declared component_type or file_format (e.g. a plot_type mismatch), producing an unreliable qmd

## Inputs

- VueGen YAML configuration file (report title, description, graphical_abstract, logo, sections/subsections/components hierarchy)
- Component output files: plots (plotly, altair, static, interactive_network), tabular data (csv, xls, txt, parquet), Markdown text, HTML components
- Requested output format (PDF, HTML, DOCX, ODT, PPTX, Reveal.js, Jupyter notebook, Streamlit)

## Outputs

- Generated Quarto Markdown file (report.qmd) with YAML metadata header and section-mirroring headings
- Rendered report document in the requested format (e.g. report.html, report.pdf, report.pptx) or notebook/web-app output

## How to apply

Load the YAML configuration (e.g. Basic_example_vuegen_demo_notebook_config.yaml) with Python's yaml parser to extract report metadata (title, description, graphical_abstract, logo) and the sections/subsections/components hierarchy. Resolve each component's file_path and validate component_type (plot, dataframe, markdown, html, apicall, chatbot) and sub-type (plot_type: plotly, altair, static, interactive_network; file_format: csv, xls, txt, parquet) against the declared files. Emit a report.qmd whose YAML header carries the title, description, logo and graphical abstract, with nested headings mirroring each section and Quarto embeds for plots, aggrid tables, networks, Markdown text, and HTML components. Render report.qmd with Quarto, invoked through the quarto-cli Python library bundled in the pip installation, into the requested target format. Finally confirm the rendered output exists, opens as a valid document of that type, and carries the config-declared title with all configured components present.

## Related tools

- **VueGen** (Automates creation of the reports from a YAML config; generates the qmd directory structure before rendering) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Quarto** (Renders the generated report.qmd into the requested static format (e.g. HTML, PDF, PPTX))
- **quarto-cli** (Python library bundled in the pip installation used to invoke Quarto rendering programmatically) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Python** (Package language used to parse the YAML configuration and orchestrate qmd generation) — https://github.com/Multiomics-Analytics-Group/vuegen

## Examples

```
quarto render report.qmd --to html
```

## Evaluation signals

- The rendered output file exists in the requested format (e.g. report.html after quarto render)
- The output opens as a valid document of the declared type
- The rendered document's title matches the report title declared in the YAML config
- All configured components (plots, aggrid tables, networks, Markdown text, HTML) are present in the rendered report
- Section/subsection headings in the output mirror the sections and subsections in the configuration

## Limitations

- Static rendering is only one report type; Streamlit web applications and Jupyter notebooks are handled as separate output types by VueGen
- Component sub-types must match declared values (plot_type: plotly, altair, static, interactive_network; file_format: csv, xls, txt, parquet); mismatches must be caught before rendering
- Quarto must be available via the quarto-cli Python library bundled in the pip installation for rendering to work

## Evidence

- [full_text] qmd structures entire report: [evidence span withheld — anchor status: not_found]
- [abstract] supported formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [abstract] report from output files: "users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format"
- [readme] tool purpose: "VueGen is a Python package that automates the creation of scientific reports."
