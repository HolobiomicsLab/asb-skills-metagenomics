---
name: hierarchical-section-component-instantiation
description: Use when a YAML report config must be turned into instantiated section/subsection/component
  objects — e.g., implementing or extending VueGen's report generators (Streamlit,
  notebook, document) for bioinformatics outputs like plots, tables, networks, Markdown,
  and HTML.
when_to_use_negative:
- The report structure is not described in a YAML config — e.g., hand-authored documents
  or notebooks where no report/section/subsection/component hierarchy exists to instantiate.
- The desired output is a raw visualization of multi-omics data rather than a structured
  report assembled from existing output files — VueCore-style plotting is the wrong
  layer.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3071
tools:
- name: VueGen
  role: Emits the report-creation logic that parses the YAML hierarchy and instantiates
    section/subsection/component classes; source of the generated Streamlit app structure.
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Streamlit
  role: 'Target web-application format: section/subsection hierarchy is mapped onto
    Streamlit''s multipage-app convention (one subfolder per section, one page script
    per subsection).'
- name: Python
  role: Implementation language used to read the YAML config, instantiate classes,
    and write the generated page scripts.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:a0db6799d939dd200b9b22647c33476fc86332557905f74aab278ca01f5e35f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Python
  - Streamlit
  - VueGen
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hierarchical-section-component-instantiation

## Summary

Parses a YAML configuration describing a report → sections → subsections → components hierarchy, instantiates the corresponding report/section/subsection classes and type-specific component objects, and maps that hierarchy onto output renderers (e.g., Streamlit multipage apps). This is the core internal logic of VueGen's report-creation pipeline.

## When to use

Use when a YAML report config must be turned into instantiated section/subsection/component objects — e.g., implementing or extending VueGen's report generators (Streamlit, notebook, document) for bioinformatics outputs like plots, tables, networks, Markdown, and HTML.

## When NOT to use

- The report structure is not described in a YAML config — e.g., hand-authored documents or notebooks where no report/section/subsection/component hierarchy exists to instantiate.
- The desired output is a raw visualization of multi-omics data rather than a structured report assembled from existing output files — VueCore-style plotting is the wrong layer.

## Inputs

- YAML configuration file describing report/section/subsection/component hierarchy (e.g., Earth_microbiome_vuegen_demo_notebook_config.yaml)
- Directory of bioinformatics output files (plots, tables, networks, Markdown text, HTML components)
- Report format specification (PDF, HTML, DOCX, ODT, PPTX, Reveal.js, Jupyter notebook, Streamlit)

## Outputs

- Instantiated report, section, and subsection class objects
- Component objects typed by component_type (plot, dataframe, markdown, html, apicall, chatbot) with attributes like plot_type, file_format, delimiter
- For Streamlit reports: a directory with one subfolder per section, one Python page script per subsection, and a main entry script streamlit_report_app.py

## How to apply

Read the YAML configuration (e.g., Earth_microbiome_vuegen_demo_notebook_config.yaml) with Python and parse the report, sections, subsections, and components hierarchy. Instantiate the report, section, and subsection classes from the parsed structure, then instantiate each component object according to its component_type (plot, dataframe, markdown, html, apicall, chatbot) and type-specific attributes such as plot_type, file_format, and delimiter. Map the section/subsection hierarchy onto the target renderer's conventions — for Streamlit, one subfolder per section and one Python page script per subsection that renders the subsection's components, assembled by a main entry script (streamlit_report_app.py) that respects the streamlit_autorun=False default. Validate the generated app locally with --report-type streamlit and check formatting/tests with black, ruff, and pytest.

## Related tools

- **VueGen** (Emits the report-creation logic that parses the YAML hierarchy and instantiates section/subsection/component classes; source of the generated Streamlit app structure.) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Streamlit** (Target web-application format: section/subsection hierarchy is mapped onto Streamlit's multipage-app convention (one subfolder per section, one page script per subsection).)
- **Python** (Implementation language used to read the YAML config, instantiate classes, and write the generated page scripts.)

## Examples

```
vuegen --config Earth_microbiome_vuegen_demo_notebook_config.yaml --report-type streamlit
```

## Evaluation signals

- Running the generated app locally with --report-type streamlit produces a working multipage app with one page per subsection.
- Generated code passes black and ruff formatting checks.
- Repository tests (pytest) pass after generating or modifying the report logic.
- Every component_type in the YAML (plot, dataframe, markdown, html, apicall, chatbot) yields a correctly typed component object with its type-specific attributes (plot_type, file_format, delimiter) populated.
- Streamlit directory structure mirrors the YAML hierarchy: one subfolder per section, one Python page script per subsection; streamlit_autorun=False default is respected.

## Limitations

- The mapping targets Streamlit's multipage-app convention; other report formats (documents, presentations, notebooks) follow different conventions and need their own renderers.
- Correctness depends on the YAML hierarchy being complete and component_type/attributes being valid; malformed configs will propagate into broken instantiation.
- streamlit_autorun defaults to False, so the generated app is not automatically launched — automation that expects auto-launch must handle this.

## Evidence

- [methods] For the Streamlit report type, VueGen emits a directory with subfolders for every section, each containing Python scripts for individual subsections that correspond to pages in the web application.: [evidence span withheld — anchor status: not_found]
- [methods] instantiate each component object according to its component_type and type-specific attributes: [evidence span withheld — anchor status: not_found]
- [abstract] VueGen produces reports from a directory of output files plus report format: "users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format"
- [methods] Contribution checks: black, ruff, and pytest are the validation gates for changes.: [evidence span withheld — anchor status: not_found]
