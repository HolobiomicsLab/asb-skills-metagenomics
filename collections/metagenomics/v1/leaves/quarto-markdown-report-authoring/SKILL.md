---
name: quarto-markdown-report-authoring
description: Use when a bioinformatics pipeline's output files (plots, tables, networks,
  Markdown text, HTML components) must be assembled into a static document or presentation
  whose structure is declared in a YAML configuration file.
when_to_use_negative:
- The desired output is an interactive Streamlit web application or a Jupyter notebook
  rather than a static document/presentation rendered from qmd
- The report content is supplied as a single flat file with no YAML-declared section/subsection/component
  hierarchy to mirror
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VueGen
  role: Generates the directory with the Quarto Markdown (.qmd) report file structured
    from the YAML configuration
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Quarto
  role: Renders the emitted report.qmd into the requested static format (PDF, HTML,
    DOCX, ODT, PPTX, Reveal.js)
- name: quarto-cli
  role: Python library bundled in the VueGen pip installation used to invoke Quarto
    rendering programmatically
- name: Python
  role: Parses the YAML configuration and drives qmd generation via the VueGen package
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:3d6eb31c7832f159988da25555a32eb0f5359452ee90422594fe50e379b28ada
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# quarto-markdown-report-authoring

## Summary

Author a structured Quarto Markdown (.qmd) report from a YAML configuration and render it with Quarto into static formats such as PDF, HTML, DOCX, ODT, PPTX, or Reveal.js, as implemented in VueGen for scientific report generation.

## When to use

Use when a bioinformatics pipeline's output files (plots, tables, networks, Markdown text, HTML components) must be assembled into a static document or presentation whose structure is declared in a YAML configuration file.

## When NOT to use

- The desired output is an interactive Streamlit web application or a Jupyter notebook rather than a static document/presentation rendered from qmd
- The report content is supplied as a single flat file with no YAML-declared section/subsection/component hierarchy to mirror

## Inputs

- VueGen YAML configuration file (report title, description, graphical_abstract, logo, sections/subsections/components)
- Output files referenced by component file_path: plots (plotly, altair, static, interactive_network), dataframes (csv, xls, txt, parquet), Markdown text, HTML components
- Target report format (e.g., PDF, HTML, DOCX, ODT, PPTX, Reveal.js)

## Outputs

- report.qmd file with YAML header (title, description, logo, graphical abstract) and section headings plus component embeds
- Rendered static report in the requested format (e.g., report.html)

## How to apply

Load the VueGen YAML configuration (e.g., Basic_example_vuegen_demo_notebook_config.yaml) with Python's yaml parser to extract report metadata (title, description, graphical_abstract, logo) and the sections/subsections/components hierarchy. Resolve each component's file_path and validate its component_type (plot, dataframe, markdown, html, apicall, chatbot) and sub-type (plot_type: plotly, altair, static, interactive_network; file_format: csv, xls, txt, parquet) against the declared files. Emit a report.qmd file whose YAML header carries the title, description, logo and graphical abstract, with nested headings mirroring each section and subsection and appropriate Quarto embeds for plots, aggrid tables, networks, Markdown text, and HTML components. Render report.qmd with Quarto — invoked through the quarto-cli Python library bundled in the pip installation — into the requested format (e.g., HTML via quarto render).

## Related tools

- **VueGen** (Generates the directory with the Quarto Markdown (.qmd) report file structured from the YAML configuration) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Quarto** (Renders the emitted report.qmd into the requested static format (PDF, HTML, DOCX, ODT, PPTX, Reveal.js))
- **quarto-cli** (Python library bundled in the VueGen pip installation used to invoke Quarto rendering programmatically)
- **Python** (Parses the YAML configuration and drives qmd generation via the VueGen package)

## Examples

```
quarto render report.qmd --to html
```

## Evaluation signals

- The rendered output file exists in the requested format (e.g., .html, .pdf, .pptx)
- The rendered file opens as a valid document of that type
- The report title in the rendered output matches the title declared in the YAML configuration
- All components configured in the YAML (plots, aggrid tables, networks, Markdown text, HTML components) are present in the rendered report
- The qmd's nested headings mirror the sections/subsections hierarchy of the configuration

## Limitations

- Static report types only cover the qmd-then-Quarto path; Jupyter notebooks and Streamlit web applications use different generation routes
- Rendering depends on the quarto-cli Python library bundled in the pip installation, so the environment must have Quarto available for the target format

## Evidence

- [abstract] Supported formats for static output: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [full_text] VueGen qmd generation and Quarto rendering: [evidence span withheld — anchor status: not_found]
- [readme] VueGen purpose from README: "VueGen is a Python package that automates the creation of scientific reports."
- [abstract] Components accepted in reports: "users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components"
