---
name: python-report-generator-implementation
description: Use when a pipeline's bioinformatics outputs (plots, tables, networks,
  Markdown text, HTML components) must be turned into an interactive Streamlit web
  application driven by a structured YAML configuration of sections, subsections,
  and component objects.
when_to_use_negative:
- The desired output is a static deliverable (PDF, HTML, DOCX, ODT document, PPTX/Reveal.js
  presentation, or Jupyter notebook) rather than a Streamlit web application
- The bioinformatics outputs exist only as loose files with no section/subsection/component
  hierarchy expressible in the YAML configuration
- The user cannot run a Python/Streamlit environment and needs the packaged desktop
  application instead of generated app code
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VueGen
  role: 'Tool whose report-creation logic is implemented: parses the YAML config,
    instantiates report/section/subsection/component classes, and emits the Streamlit
    app'
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Streamlit
  role: Target web-application framework whose multipage-app convention dictates the
    subfolder-per-section, page-script-per-subsection layout
- name: Python
  role: Implementation language for the config parser, component classes, and generated
    page scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:a07ce1e2620e0e71274e2cd91a590f7d2ea75ea10abfe27ae941b3e0c0aa5422
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Python
  - Streamlit
  - VueGen
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# python-report-generator-implementation

## Summary

Implements the report-creation logic of a Python package (as in VueGen) that parses a YAML configuration hierarchy of report, sections, subsections, and components and emits executable report artefacts — here a multipage Streamlit web application with one subfolder per section and one page script per subsection.

## When to use

Use when a pipeline's bioinformatics outputs (plots, tables, networks, Markdown text, HTML components) must be turned into an interactive Streamlit web application driven by a structured YAML configuration of sections, subsections, and component objects.

## When NOT to use

- The desired output is a static deliverable (PDF, HTML, DOCX, ODT document, PPTX/Reveal.js presentation, or Jupyter notebook) rather than a Streamlit web application
- The bioinformatics outputs exist only as loose files with no section/subsection/component hierarchy expressible in the YAML configuration
- The user cannot run a Python/Streamlit environment and needs the packaged desktop application instead of generated app code

## Inputs

- YAML configuration file describing the report, sections, subsections, and components hierarchy (e.g. Earth_microbiome_vuegen_demo_notebook_config.yaml)
- Directory of pipeline output files referenced by components: plots, tables, networks, Markdown text, and HTML components
- Component metadata per component_type (plot, dataframe, markdown, html, apicall, chatbot) including plot_type, file_format, and delimiter

## Outputs

- Multipage Streamlit application directory with one subfolder per section
- One Python page script per subsection that renders the subsection's components
- Main entry script streamlit_report_app.py that assembles the pages

## How to apply

Read the YAML configuration file (e.g. Earth_microbiome_vuegen_demo_notebook_config.yaml) with Python and parse the report, sections, subsections, and components hierarchy. Instantiate the report, section, and subsection classes from the parsed structure, then instantiate each component object according to its component_type — plot, dataframe, markdown, html, apicall, chatbot — respecting type-specific attributes such as plot_type, file_format, and delimiter. Map the section/subsection hierarchy onto Streamlit's multipage-app convention: create one subfolder per section and one Python page script per subsection that renders that subsection's components. Write the main entry script (streamlit_report_app.py) that assembles the pages and honours the streamlit_autorun=False default. Finally, validate the generated app by running it locally with --report-type streamlit and checking formatting and tests with black, ruff, and pytest.

## Related tools

- **VueGen** (Tool whose report-creation logic is implemented: parses the YAML config, instantiates report/section/subsection/component classes, and emits the Streamlit app) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Streamlit** (Target web-application framework whose multipage-app convention dictates the subfolder-per-section, page-script-per-subsection layout)
- **Python** (Implementation language for the config parser, component classes, and generated page scripts)

## Examples

```
vuegen --report-type streamlit --config Earth_microbiome_vuegen_demo_notebook_config.yaml
```

## Evaluation signals

- Generated app runs locally when invoked with --report-type streamlit and renders without errors
- Output directory structure contains exactly one subfolder per section and one Python page script per subsection
- All component objects are instantiated with the correct type-specific attributes (plot_type, file_format, delimiter) for their component_type
- The main entry script streamlit_report_app.py assembles all pages and respects the streamlit_autorun=False default
- Formatting and tests pass: black and ruff checks are clean and pytest tests run successfully

## Limitations

- Requires a well-formed YAML configuration with a resolvable report/section/subsection/component hierarchy; malformed configs break instantiation
- Only component types the implementation knows (plot, dataframe, markdown, html, apicall, chatbot) are supported; other output content cannot be rendered
- Quality of the generated app depends on contributions passing black, ruff, and pytest checks before merge

## Evidence

- [abstract] Supported report formats span Streamlit apps: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [abstract] Users supply output files and a report format: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [methods] Contribution validation with black, ruff, and pytest: [evidence span withheld — anchor status: not_found]
- [readme] VueGen is a Python report-automation package: "VueGen is a Python package that automates the creation of scientific reports."
