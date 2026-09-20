---
name: network-edgelist-static-rendering
description: 'Use when a VueGen report configuration declares a component with component_type:
  network, plot_type: interactive_network, and csv_network_format: EDGELIST, and the
  target report type is a static format such as PDF, DOCX, ODT, PPTX, or Reveal.js.'
when_to_use_negative:
- The target report type is interactive (HTML, Streamlit, or Jupyter), where the interactive
  network should be preserved rather than rasterized
- 'The network component is already flagged plot_type: static and requires no interactive-to-static
  conversion'
- The component is a dataframe (csv, xls, txt, parquet, xlsx) rather than a network
  — dataframes follow separate static/interactive table rendering rules
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VueGen
  role: Python package whose report-generation pipeline performs interactive-to-static
    conversion of network components based on the report configuration YAML
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Python
  role: Implementation language for the conversion module (PEP 484 type hints, NumPy-style
    docstrings, black/ruff formatting, pytest tests)
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Git
  role: Used to fork, branch, and submit pull requests for the conversion implementation
    to the vuegen repository
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:175b8950273770a15e12dd00303448c64d824b69ac069cf7cce764f9a6584ce1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Git
  - Python
  - VueGen
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# network-edgelist-static-rendering

## Summary

Renders interactive network components declared in a VueGen report configuration as static network images when the target report format is static (PDF, DOCX, ODT, PPTX, Reveal.js), building the graph from an EDGELIST-format CSV.

## When to use

Use when a VueGen report configuration declares a component with component_type: network, plot_type: interactive_network, and csv_network_format: EDGELIST, and the target report type is a static format such as PDF, DOCX, ODT, PPTX, or Reveal.js.

## When NOT to use

- The target report type is interactive (HTML, Streamlit, or Jupyter), where the interactive network should be preserved rather than rasterized
- The network component is already flagged plot_type: static and requires no interactive-to-static conversion
- The component is a dataframe (csv, xls, txt, parquet, xlsx) rather than a network — dataframes follow separate static/interactive table rendering rules

## Inputs

- Report configuration YAML with component_type, plot_type, file_format, and csv_network_format fields
- Network component CSV in EDGELIST format (csv_network_format: EDGELIST)
- Target report type from the supported list (PDF, DOCX, ODT, PPTX, Reveal.js, HTML, Streamlit, Jupyter)

## Outputs

- Static network image embedded in the generated static report (PDF, DOCX, ODT, PPTX, or Reveal.js)

## How to apply

Parse the report configuration YAML to classify the component via its component_type, plot_type, and csv_network_format fields, and read the target report type from the supported list (PDF, DOCX, ODT, PPTX, Reveal.js for static; HTML, Streamlit, Jupyter for interactive). If the target is static and the network is declared plot_type: interactive_network with csv_network_format: EDGELIST, build the graph from the edge-list CSV and render it to a static network image. Leave components already flagged plot_type: static untouched. Implement the conversion module with PEP 484 type hints and NumPy-style docstrings, formatted with black and linted with ruff, and add independent pytest tests covering typical use cases and edge cases such as empty inputs.

## Related tools

- **VueGen** (Python package whose report-generation pipeline performs interactive-to-static conversion of network components based on the report configuration YAML) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Python** (Implementation language for the conversion module (PEP 484 type hints, NumPy-style docstrings, black/ruff formatting, pytest tests)) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Git** (Used to fork, branch, and submit pull requests for the conversion implementation to the vuegen repository) — https://github.com/Multiomics-Analytics-Group/vuegen

## Examples

```
pytest .  # from the vuegen repository root; validates the network EDGELIST static-rendering component-handling tests
```

## Evaluation signals

- Run `pytest .` from the repository root; all component-handling tests pass locally before any PR submission
- Static report targets receive a rendered static network image instead of an interactive network widget
- Components already flagged plot_type: static remain untouched by the conversion step
- Changes pass `black` and `ruff` checks as required by the vuegen contribution workflow
- Edge cases such as empty edge-list CSV inputs are covered by independent, locally runnable tests in tests/ mirroring the source structure

## Limitations

- Conversion is only defined for networks declared with plot_type: interactive_network and csv_network_format: EDGELIST; other network CSV layouts are not covered by this workflow
- Static rendering rasterizes the network, losing interactivity in document-style reports (PDF, DOCX, ODT, PPTX, Reveal.js)
- The README and article excerpts are truncated, so exact rendering parameters and image format details must be confirmed in the vuegen source

## Evidence

- [methods] Interactive networks converted for static targets: [evidence span withheld — anchor status: not_found]
- [abstract] Supported static report formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [methods] Contribution checks required before PR: [evidence span withheld — anchor status: not_found]
- [abstract] Tool scope: "We present VueGen, a tool that automates the creation of reports from bioinformatics outputs, allowing researchers with minimal coding experience"
