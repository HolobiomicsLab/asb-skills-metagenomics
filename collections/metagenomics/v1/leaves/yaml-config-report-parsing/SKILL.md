---
name: yaml-config-report-parsing
description: Use when a user supplies a VueGen YAML config (e.g. Earth_microbiome_vuegen_demo_notebook_config.yaml)
  and needs the report/section/subsection/component tree materialized as Python objects
  before rendering a report in any supported format.
when_to_use_negative:
- The input is not a VueGen YAML configuration but raw output files without a report/section/subsection/component
  structure — VueGen requires the config to define the hierarchy
- The config references component files (plot, dataframe, network) whose file_path
  does not resolve in the supplied directory, so validation would fail before generation
- The goal is rendering an already-parsed report (e.g. running quarto render or launching
  the Streamlit app) rather than building the in-memory hierarchy
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VueGen
  role: Defines the YAML config schema and provides the report/section/subsection/component
    classes instantiated from the parsed configuration
  repo: https://github.com/Multiomics-Analytics-Group/vuegen
- name: Python
  role: Language used with the yaml parser to load the config and instantiate the
    object hierarchy
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf149
    title: 'VueGen: automating the generation of scientific reports'
  source_content_hash: sha256:0dbd464ce65b72b8fced6521fb373a84cf2a2c4a930a16ffb962ecada5c053c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Multiomics-Analytics-Group/vuegen
  tools:
  - Jupyter
  - Python
  - VueGen
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# yaml-config-report-parsing

## Summary

Parse a VueGen YAML configuration file into its report → sections → subsections → components hierarchy and instantiate the corresponding Python objects, so the parsed structure can drive generation of Streamlit, Quarto, or notebook reports from bioinformatics outputs.

## When to use

Use when a user supplies a VueGen YAML config (e.g. Earth_microbiome_vuegen_demo_notebook_config.yaml) and needs the report/section/subsection/component tree materialized as Python objects before rendering a report in any supported format.

## When NOT to use

- The input is not a VueGen YAML configuration but raw output files without a report/section/subsection/component structure — VueGen requires the config to define the hierarchy
- The config references component files (plot, dataframe, network) whose file_path does not resolve in the supplied directory, so validation would fail before generation
- The goal is rendering an already-parsed report (e.g. running quarto render or launching the Streamlit app) rather than building the in-memory hierarchy

## Inputs

- VueGen YAML configuration file (e.g. Basic_example_vuegen_demo_notebook_config.yaml or Earth_microbiome_vuegen_demo_notebook_config.yaml)
- Directory of bioinformatics output files (plots, tables, networks, Markdown text, HTML components) in formats such as csv, xls, txt, parquet

## Outputs

- Instantiated report, section, and subsection objects forming the report hierarchy
- Component objects typed by component_type (plot, dataframe, markdown, html, apicall, chatbot) with resolved file_path and type-specific attributes

## How to apply

Load the YAML file with Python's yaml parser and walk the hierarchy to extract report metadata (title, description, graphical_abstract, logo) plus the sections, subsections, and components underneath. Instantiate the report, section, and subsection classes from the parsed structure, and instantiate each component object according to its component_type (plot, dataframe, markdown, html, apicall, chatbot) and type-specific attributes such as plot_type (plotly, altair, static, interactive_network), file_format (csv, xls, txt, parquet), and delimiter. Resolve each component's file_path against the declared directory of outputs to validate that the referenced plot, table, network, Markdown text, or HTML component exists before report generation proceeds. The rationale is that VueGen produces reports simply from a directory of output files plus this config, so correct parsing of the hierarchy and per-component attributes is what guarantees the downstream Streamlit multipage app or Quarto qmd report matches the user's specification.

## Related tools

- **VueGen** (Defines the YAML config schema and provides the report/section/subsection/component classes instantiated from the parsed configuration) — https://github.com/Multiomics-Analytics-Group/vuegen
- **Python** (Language used with the yaml parser to load the config and instantiate the object hierarchy)

## Examples

```
import yaml; cfg = yaml.safe_load(open('Earth_microbiome_vuegen_demo_notebook_config.yaml'))  # then instantiate report/sections/subsections/components from cfg
```

## Evaluation signals

- Every section and subsection in the YAML maps to exactly one instantiated object, and every component is typed as one of plot, dataframe, markdown, html, apicall, chatbot
- Each component's file_path resolves to an existing file whose extension matches the declared file_format (csv, xls, txt, parquet)
- Report metadata (title, description, graphical_abstract, logo) extracted from the config matches what downstream renderers (Quarto YAML header, Streamlit main script) display
- For Streamlit output, the parsed hierarchy yields one subfolder per section and one Python page script per subsection; streamlit_autorun defaults to False
- Generated code passes black and ruff checks and the repository's pytest tests

## Limitations

- Component sub-types are constrained (plot_type: plotly, altair, static, interactive_network; file_format: csv, xls, txt, parquet); configs declaring other values will not map to a supported component
- Parsing correctness alone does not guarantee a valid report — rendering (e.g. via Quarto through the quarto-cli Python library) can still fail if the referenced files are malformed
- VueGen assumes users supply a directory of outputs plus the report format; it does not construct outputs themselves

## Evidence

- [other] config hierarchy: [evidence span withheld — anchor status: not_found]
- [other] component instantiation: [evidence span withheld — anchor status: not_found]
- [abstract] config-driven reports: "With VueGen, users can produce reports by simply specifying a directory containing output files, such as plots, tables, networks, Markdown text, and HTML components, along with the report format."
- [other] component validation: [evidence span withheld — anchor status: not_found]
