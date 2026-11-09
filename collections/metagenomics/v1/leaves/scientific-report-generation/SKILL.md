---
name: scientific-report-generation
description: Use when a pipeline or analysis has produced intermediate output files
  (plots, dataframes, networks, Markdown, HTML components) and a shareable report
  in a specific format must be generated from them with minimal coding.
when_to_use_negative:
- The input is already a finished report document and no re-rendering into another
  format is needed
- The target output is a raw data file or feature table rather than a human-readable
  report — VueGen renders reports, not data transformations
- Quarto (or the runtime for the chosen report type, e.g. Streamlit/Jupyter) cannot
  be installed in the execution environment and only document/presentation formats
  are required
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3071
tools:
- name: VueGen
  role: 'Primary tool: automates creation of reports from bioinformatics outputs;
    invoked via CLI with --config/--directory and --report-type flags and rendered
    via Quarto'
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Quarto
  role: Rendering backend used by VueGen to build document and presentation report
    formats (PDF, HTML, DOCX, ODT, PPTX, Reveal.js)
- name: Python
  role: Language/runtime in which VueGen is distributed as a package; used to install
    and run the CLI
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Docker
  role: Containerized distribution channel for running VueGen reproducibly
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: nf-core
  role: VueGen is packaged as an nf-core module that runs as the final stage of a
    Nextflow workflow, collecting upstream process outputs into a report
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Streamlit
  role: 'Target report format: VueGen can generate Streamlit web applications (with
    streamlit_autorun left at its default False)'
- name: Jupyter
  role: 'Target report format: VueGen can generate Jupyter notebooks as reports'
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:dbec606fa8f1f88b385ed932604f155d3319dc2a18aa6685a7802ce9a1c880d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Docker
  - Jupyter
  - Python
  - Quarto
  - Streamlit
  - VueGen
  - nf-core
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# scientific-report-generation

## Summary

Automates the creation of scientific reports from bioinformatics output files using VueGen, which assembles plots, tables, networks, Markdown text, and HTML components into reports rendered via Quarto. It supports documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications.

## When to use

Use when a pipeline or analysis has produced intermediate output files (plots, dataframes, networks, Markdown, HTML components) and a shareable report in a specific format must be generated from them with minimal coding.

## When NOT to use

- The input is already a finished report document and no re-rendering into another format is needed
- The target output is a raw data file or feature table rather than a human-readable report — VueGen renders reports, not data transformations
- Quarto (or the runtime for the chosen report type, e.g. Streamlit/Jupyter) cannot be installed in the execution environment and only document/presentation formats are required

## Inputs

- A directory of bioinformatics output files (plots, tables, networks, Markdown text, HTML components)
- A YAML config file specifying report structure: sections, subsections, and components with component_type/file_path entries
- A report-type value (Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, Jupyter)
- A Quarto installation (required for document/presentation report types)

## Outputs

- A rendered report in one of: PDF, HTML, DOCX, ODT, PPTX, Reveal.js, Jupyter notebook, or Streamlit web application
- A report directory artifact (e.g. 'vuegen_basic_case_study_report.html') consumable by downstream aggregation tools as an HTML component

## How to apply

First, choose the invocation mode: install VueGen as a Python package via pip from the GitHub repository, use the Docker image, call the nf-core module as the final stage of a Nextflow workflow, or use the desktop application. Next, supply either a YAML config specifying 'the structure of the report, including sections, subsections, and various components' (with component_type/file_path entries for plot, dataframe, markdown, html, apicall, chatbot components), or simply point the CLI at a directory of outputs with the '--directory' flag. Then select the report type with '--report-type' from the supported set: Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, or Jupyter; ensure Quarto is available for document and presentation formats. For example, run the CLI with '--directory example_data/Basic_example_vuegen_demo_notebook --report-type html' to reproduce Case Study 1 and save the rendered output (e.g. 'vuegen_basic_case_study_report.html'). When embedding in Nextflow, stage collected upstream files into a flat working directory, generate the VueGen-compatible YAML, execute 'vuegen --config <config.yaml> --report-type <type>' with streamlit_autorun at its default False, and emit the report directory as the process output channel for downstream consumers such as MultiQC/AssemblyQC-style HTML reports.

## Related tools

- **VueGen** (Primary tool: automates creation of reports from bioinformatics outputs; invoked via CLI with --config/--directory and --report-type flags and rendered via Quarto) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Quarto** (Rendering backend used by VueGen to build document and presentation report formats (PDF, HTML, DOCX, ODT, PPTX, Reveal.js))
- **Python** (Language/runtime in which VueGen is distributed as a package; used to install and run the CLI) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Docker** (Containerized distribution channel for running VueGen reproducibly) — https://github.com/Multiomics-Analytics-Group/vuegen
- **nf-core** (VueGen is packaged as an nf-core module that runs as the final stage of a Nextflow workflow, collecting upstream process outputs into a report) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Streamlit** (Target report format: VueGen can generate Streamlit web applications (with streamlit_autorun left at its default False))
- **Jupyter** (Target report format: VueGen can generate Jupyter notebooks as reports)

## Examples

```
vuegen --directory example_data/Basic_example_vuegen_demo_notebook --report-type html
```

## Evaluation signals

- The VueGen process executes successfully on the staged directory and produces a report artifact in the requested format (paper reports no quantitative benchmark for this step; acceptance is qualitative)
- The rendered report contains the expected components matching the YAML config or directory contents (plots, dataframes, networks, Markdown, HTML components) organized into the specified sections and subsections
- For Case Study 1 reproduction, the output 'vuegen_basic_case_study_report.html' is generated from the predefined 'example_data/Basic_example_vuegen_demo_notebook' directory
- The emitted report directory is a valid HTML component that downstream tools (e.g. MultiQC/AssemblyQC-style reports) can consume
- main.nf of the nf-core module contains a process that runs the vuegen CLI with a --report-type flag

## Limitations

- No quantitative benchmark is reported for report generation; validation of the nf-core module step is qualitative (the process must execute vuegen successfully and produce a report artifact)
- Streamlit reports default to not auto-running (streamlit_autorun=False), so an extra launch step may be needed to serve the app
- Quarto must be available in the environment for document/presentation report types, otherwise those formats will fail
- Component files must match VueGen-supported component types (plot, dataframe, markdown, html, apicall, chatbot); unsupported outputs need conversion first

## Evidence

- [abstract] abstract: VueGen automates report creation from output files: "We present VueGen, a tool that automates the creation of reports from bioinformatics outputs, allowing researchers with minimal coding experience to communicate their results effectively."
- [abstract] abstract: supported report formats: "Supported formats include documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications."
- [abstract] abstract: inputs to report generation: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [other] task_001 workflow: CLI invocation for Case Study 1: [evidence span withheld — anchor status: not_found]
- [other] task_006 workflow: nf-core module configuration: [evidence span withheld — anchor status: not_found]
- [other] task_006 workflow: supported report types list: "The current report types supported by VueGen are: Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, Jupyter"
