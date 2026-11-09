---
name: streamlit-multi-page-app-generation
description: Use when a YAML report configuration defines sections, subsections, and
  components (plot, dataframe, markdown, html, apicall, chatbot) and the desired output
  format is an interactive web application rather than a document, presentation, or
  notebook.
when_to_use_negative:
- The required output is a static document (PDF, HTML, DOCX, ODT), presentation (PPTX,
  Reveal.js), or Jupyter notebook — use the corresponding VueGen report type instead
- There is no structured report hierarchy (sections/subsections/components) to map
  onto pages — a hand-written single-page Streamlit script would be simpler
- Inputs are raw bioinformatics outputs that have not yet been described in a VueGen-style
  configuration directory
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3077
tools:
- name: VueGen
  role: Parses the YAML configuration, instantiates the report/section/subsection/component
    classes, and emits the Streamlit multipage app structure
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Streamlit
  role: Target web-application framework whose multipage-app convention (subfolder
    per section, page script per subsection) the generated code follows
- name: Python
  role: Language in which the YAML is parsed, classes instantiated, and the generated
    page scripts written
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:555aeb9cb11317415a1f3f2ec467afbede7169a7c023f5865b704fb47b9ad9e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Python
  - Streamlit
  - VueGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Streamlit multi-page app generation

## Summary

Generates a multipage Streamlit web application from a parsed report hierarchy, emitting one subfolder per section and one Python page script per subsection that renders the subsection's components. This is the Streamlit report path of VueGen's report-creation logic.

## When to use

Use when a YAML report configuration defines sections, subsections, and components (plot, dataframe, markdown, html, apicall, chatbot) and the desired output format is an interactive web application rather than a document, presentation, or notebook.

## When NOT to use

- The required output is a static document (PDF, HTML, DOCX, ODT), presentation (PPTX, Reveal.js), or Jupyter notebook — use the corresponding VueGen report type instead
- There is no structured report hierarchy (sections/subsections/components) to map onto pages — a hand-written single-page Streamlit script would be simpler
- Inputs are raw bioinformatics outputs that have not yet been described in a VueGen-style configuration directory

## Inputs

- YAML report configuration file (e.g. Earth_microbiome_vuegen_demo_notebook_config.yaml) defining report, sections, subsections, components
- Directory of output files such as plots, tables, networks, Markdown text, and HTML components
- Component attributes: component_type (plot, dataframe, markdown, html, apicall, chatbot), plot_type, file_format, delimiter

## Outputs

- Streamlit multipage app directory with one subfolder per section
- One Python page script per subsection, each rendering the subsection's components
- Main entry script streamlit_report_app.py assembling the pages

## How to apply

First parse the YAML configuration file (e.g. Earth_microbiome_vuegen_demo_notebook_config.yaml) into the report > sections > subsections > components hierarchy. Instantiate the report, section, and subsection classes from the parsed structure, and instantiate each component object according to its component_type (plot, dataframe, markdown, html, apicall, chatbot) together with type-specific attributes such as plot_type, file_format, and delimiter. Map the section/subsection hierarchy onto Streamlit's multipage-app convention: create one subfolder per section and one Python page script per subsection that renders that subsection's components. Write a main entry script (streamlit_report_app.py) that assembles the pages and honors the streamlit_autorun=False default. Finally, validate by running the app locally with --report-type streamlit and verifying formatting and tests pass black, ruff, and pytest.

## Related tools

- **VueGen** (Parses the YAML configuration, instantiates the report/section/subsection/component classes, and emits the Streamlit multipage app structure) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Streamlit** (Target web-application framework whose multipage-app convention (subfolder per section, page script per subsection) the generated code follows)
- **Python** (Language in which the YAML is parsed, classes instantiated, and the generated page scripts written)

## Examples

```
vuegen --report-type streamlit
```

## Evaluation signals

- Generated directory tree contains exactly one subfolder per section and one Python page script per subsection matching the YAML hierarchy
- Every component in the configuration (plot, dataframe, markdown, html, apicall, chatbot) appears as rendered content in its subsection page
- streamlit_report_app.py assembles all pages and respects streamlit_autorun=False
- App launches locally via `vuegen --report-type streamlit` and pages render correctly
- `black` and `ruff` checks pass and `pytest` test suite passes on the generated code

## Limitations

- The streamlit_autorun default is False, so the app must be launched manually by the user
- Generation depends on correctly specified component_type and type-specific attributes (plot_type, file_format, delimiter); a wrong attribute breaks the rendered page
- Only output files supported by VueGen (plots, tables, networks, Markdown text, HTML components) can be embedded in pages

## Evidence

- [other] Streamlit emission pattern: [evidence span withheld — anchor status: not_found]
- [abstract] Supported report formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [abstract] Input file types: "users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components"
- [methods] Quality gates: [evidence span withheld — anchor status: not_found]
