---
name: multi-component-document-assembly
description: Use when a directory of mixed bioinformatics output files (plots, tables,
  networks, Markdown, HTML components) must be turned into a rendered report in a
  specified format, and the goal is automated, reproducible report generation.
when_to_use_negative:
- The outputs are a single homogeneous file type with no mixed components (plots +
  tables + text), where a simple export or template suffices without directory-based
  assembly
- The goal is interactive multi-omics visualization rather than report assembly —
  VueCore is the paper's companion package for interactive and static visualizations
  of multi-omics data
- No Quarto-supported target environment is available and only raw component files
  are needed for downstream programmatic use rather than a human-readable report
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3077
tools:
- name: VueGen
  role: Primary tool that automates report creation by reading a directory of output
    files plus a report format and rendering via Quarto
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Quarto
  role: Rendering engine invoked by VueGen to build the final report from the assembled
    components
- name: Python
  role: Language in which VueGen is distributed as a package; used to run the CLI
    or notebook workflow
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Streamlit
  role: Supported web-application report format that VueGen can target from the same
    component directory
- name: Jupyter
  role: Supported notebook report format that VueGen can target from the same component
    directory
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:56adc424de0a54e724ad897023a138c2ad8b75a9d846f9ae345f5134ff8fded8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Jupyter
  - Python
  - Quarto
  - Streamlit
  - VueGen
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-component-document-assembly

## Summary

Assembles heterogeneous scientific outputs (plots, dataframes, networks, Markdown text, and HTML components) into a single rendered report (e.g., HTML, PDF, DOCX, PPTX) using VueGen's directory-driven workflow built on Quarto, as demonstrated in the paper's predefined-directory case study.

## When to use

Use when a directory of mixed bioinformatics output files (plots, tables, networks, Markdown, HTML components) must be turned into a rendered report in a specified format, and the goal is automated, reproducible report generation.

## When NOT to use

- The outputs are a single homogeneous file type with no mixed components (plots + tables + text), where a simple export or template suffices without directory-based assembly
- The goal is interactive multi-omics visualization rather than report assembly — VueCore is the paper's companion package for interactive and static visualizations of multi-omics data
- No Quarto-supported target environment is available and only raw component files are needed for downstream programmatic use rather than a human-readable report

## Inputs

- Directory of bioinformatics output files containing plots, dataframes/tables, networks, Markdown text, and HTML components
- Report format specification (e.g., html, pdf, docx, odt, pptx, revealjs, notebook, streamlit)
- Predefined example directory: example_data/Basic_example_vuegen_demo_notebook

## Outputs

- Rendered report file (e.g., vuegen_basic_case_study_report.html)
- Reports in document formats (PDF, HTML, DOCX, ODT), presentation formats (PPTX, Reveal.js), Jupyter notebooks, or Streamlit web applications

## How to apply

Install VueGen from the GitHub repository (pip or the provided Docker image/nf-core module). Organize or identify a directory containing the component files — plots, dataframes/tables, network files, Markdown text blocks, and HTML components — such as the example 'example_data/Basic_example_vuegen_demo_notebook'. Decide the target output format (VueGen supports documents in PDF, HTML, DOCX, ODT; presentations in PPTX and Reveal.js; plus Jupyter notebooks and Streamlit web applications) and pass it with the directory to the VueGen CLI, e.g. via the '--directory' and '--report-type' flags. VueGen renders the report through Quarto, so the engine assembles each registered component into sectioned, format-appropriate output. Verify success by opening the rendered file (e.g., 'vuegen_basic_case_study_report.html') and confirming every component type appears correctly and interactive elements survive for HTML/notebook targets.

## Related tools

- **VueGen** (Primary tool that automates report creation by reading a directory of output files plus a report format and rendering via Quarto) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Quarto** (Rendering engine invoked by VueGen to build the final report from the assembled components)
- **Python** (Language in which VueGen is distributed as a package; used to run the CLI or notebook workflow) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Streamlit** (Supported web-application report format that VueGen can target from the same component directory)
- **Jupyter** (Supported notebook report format that VueGen can target from the same component directory)

## Examples

```
vuegen --directory example_data/Basic_example_vuegen_demo_notebook --report-type html
```

## Evaluation signals

- The rendered report file is produced without errors (e.g., vuegen_basic_case_study_report.html exists and is non-trivial in size)
- All component types from the source directory (plots, dataframes, networks, Markdown text, HTML components) appear in the rendered report
- The output matches the requested --report-type (e.g., HTML file for 'html'; DOCX/PDF/PPTX respectively for other targets)
- Re-running the CLI on the same directory yields a reproducible report consistent with prior runs (deterministic assembly)
- Interactive components (e.g., HTML widgets) remain functional when targeting HTML, notebook, or Streamlit formats

## Limitations

- Report generation depends on Quarto being available in the environment; rendering fails if the engine or its dependencies are missing
- Components must conform to the directory structure VueGen expects; arbitrarily organized output files may not be picked up correctly
- Static document targets (PDF, DOCX, ODT) cannot carry interactive HTML components, so fidelity is format-dependent

## Evidence

- [abstract] Directory-driven assembly across component types: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [abstract] Supported output formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [abstract] Two case studies showcase functionality: "To showcase VueGen's functionality, we present two case studies and provide detailed documentation to help users generate customized reports."
- [other] Case study 1 workflow with predefined directory and CLI flags: [evidence span withheld — anchor status: not_found]
- [other] Example directory contents for the basic case study: [evidence span withheld — anchor status: not_found]
