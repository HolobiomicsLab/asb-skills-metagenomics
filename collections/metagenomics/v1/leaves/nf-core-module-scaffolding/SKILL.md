---
name: nf-core-module-scaffolding
description: Use when a pipeline's terminal step must collect upstream process outputs
  (plots, tables, Markdown, HTML components) from a directory and regenerate them
  as a report, and the tool is distributed as an nf-core module.
when_to_use_negative:
- The upstream outputs are not VueGen component files (plots, tables, networks, Markdown
  text, HTML components) and cannot be described by a component_type/file_path YAML
  config
- The pipeline step is not a terminal reporting stage but an intermediate transformation
  of data
- You only need a single report generated manually, not a reproducible containerized
  workflow step
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0768
tools:
- name: VueGen
  role: Tool wrapped by the scaffolded nf-core module; its CLI generates the report
    from the staged directory and YAML config
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: nf-core
  role: Module convention (main.nf, meta.yml, container config) that the scaffold
    follows
- name: Docker
  role: Container packaging mechanism for the VueGen distribution used inside the
    process
- name: Python
  role: Language in which VueGen is distributed and invoked inside the Nextflow process
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:8aaa60cc1d619c4efba10912075bb7a7e6c3a9533c691380ad0f79cd0e731c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Docker
  - Jupyter
  - Python
  - VueGen
  - nf-core
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0768
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nf-core-module-scaffolding

## Summary

Scaffold a Nextflow process following the nf-core module convention (main.nf, meta.yml, container config) so that a Python tool such as VueGen can be invoked as the final report-generation stage of a bioinformatics pipeline.

## When to use

Use when a pipeline's terminal step must collect upstream process outputs (plots, tables, Markdown, HTML components) from a directory and regenerate them as a report, and the tool is distributed as an nf-core module.

## When NOT to use

- The upstream outputs are not VueGen component files (plots, tables, networks, Markdown text, HTML components) and cannot be described by a component_type/file_path YAML config
- The pipeline step is not a terminal reporting stage but an intermediate transformation of data
- You only need a single report generated manually, not a reproducible containerized workflow step

## Inputs

- Directory of intermediate output files from upstream Nextflow processes (plot, dataframe, markdown, html, apicall, chatbot component files)
- Report type value (one of Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, Jupyter)
- VueGen-compatible YAML config specifying report structure (sections, subsections, components)

## Outputs

- Generated report directory in the requested format (e.g. HTML, PDF, DOCX, PPTX, Jupyter notebook, Streamlit app)
- Nextflow output channel emitting the report artifact for downstream consumption

## How to apply

Scaffold the module directory with main.nf, meta.yml, and container configuration following the nf-core module convention. Declare a value channel for the report type (Streamlit, HTML, PDF, DOCX, ODT, Reveal.js, PPTX, Jupyter) and a path channel tuple-chaining component files from upstream processes, mirroring the YAML config's component_type/file_path entries. In the process script, stage collected files into a flat working directory, generate a VueGen-compatible YAML config describing sections, subsections, and components, then run the VueGen CLI with --config and --report-type (streamlit_autorun left at its default False), ensuring Quarto is available for document/presentation outputs. Emit the report directory as the output channel so downstream aggregators such as MultiQC-style HTML reports can consume it.

## Related tools

- **VueGen** (Tool wrapped by the scaffolded nf-core module; its CLI generates the report from the staged directory and YAML config) — https://github.com/Multiomics-Analytics-Group/vuegen
- **nf-core** (Module convention (main.nf, meta.yml, container config) that the scaffold follows)
- **Docker** (Container packaging mechanism for the VueGen distribution used inside the process)
- **Python** (Language in which VueGen is distributed and invoked inside the Nextflow process)

## Examples

```
vuegen --config <config.yaml> --report-type <type>
```

## Evaluation signals

- main.nf exists and contains a process that runs the vuegen CLI with a --report-type flag
- The process executes vuegen successfully on the staged directory and produces a report artifact
- The emitted report directory is consumable by downstream aggregation tools as an HTML component
- The generated YAML config correctly enumerates component_type/file_path entries matching the staged files
- Quarto is available in the container for document/presentation report types

## Limitations

- Acceptance is qualitative: the paper reports no quantitative benchmark for this step, so success is judged by whether vuegen executes and produces a report
- Report formats are limited to those VueGen supports: documents (PDF, HTML, DOCX, ODT), presentations (PPTX, Reveal.js), Jupyter notebooks, and Streamlit web applications
- Component files must be stageable into a flat working directory and describable by the YAML config's structure

## Evidence

- [abstract] VueGen is distributed as a Python package, Docker image, nf-core module, and desktop application.: "VueGen is distributed as a Python package, Docker image, nf-core module, and desktop application."
- [abstract] VueGen produces reports by specifying a directory containing output files: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [readme] VueGen is a Python package that automates the creation of scientific reports.: "VueGen is a Python package that automates the creation of scientific reports."
