---
name: command-line-bioinformatics-pipeline-execution
description: Use when a workflow step requires running a repository-hosted command-line
  tool (e.g. PICRUSt2/MinPath) on a KO abundance table to produce a pathway abundance
  table.
when_to_use_negative:
- Input is already a pathway abundance table rather than a KO abundance table — no
  inference step is needed
- The goal is direct sequencing-based functional profiling rather than prediction
  of metagenome functions from amplicon data
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_3174
tools:
- name: PICRUSt2
  role: The parent command-line pipeline whose installation wiki and bundled example
    data drive the workflow; installation instructions are documented on its GitHub
    wiki
  repo: https://github.com/picrust/picrust2
- name: MinPath
  role: Pathway-inference component that performs minimum-parsimony reconstruction
    over KEGG/MetaCyc pathway databases to map KO abundances to pathway abundances
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:9b0be13dd667b9f6294aa11fa21c6f2c63ed718fec36d098f8a748698529c908
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust2
  tools:
  - MinPath
  - PICRUSt2
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# command-line-bioinformatics-pipeline-execution

## Summary

Executes a bioinformatics workflow (here, PICRUSt2's MinPath pathway inference) by installing a tool from its documented repository and running it from the command line on tabular inputs, writing structured output files such as path_abun.tsv.

## When to use

Use when a workflow step requires running a repository-hosted command-line tool (e.g. PICRUSt2/MinPath) on a KO abundance table to produce a pathway abundance table.

## When NOT to use

- Input is already a pathway abundance table rather than a KO abundance table — no inference step is needed
- The goal is direct sequencing-based functional profiling rather than prediction of metagenome functions from amplicon data

## Inputs

- KO abundance table (e.g. the example KO abundance file included with the PICRUSt2 repository)
- KEGG/MetaCyc pathway reference databases

## Outputs

- path_abun.tsv — a tab-delimited table with pathway identifiers and abundance columns

## How to apply

Install the tool following its official wiki installation instructions (https://github.com/picrust/picrust2/wiki/Installation), then load the example KO abundance file bundled with the PICRUSt2 repository as input to the pathway-inference step. Run MinPath, which performs minimum-parsimony reconstruction over the KEGG/MetaCyc pathway databases, to map KO abundances to pathway abundances. Write the reconstructed pathway abundance table to path_abun.tsv. Because MinPath is distributed inside the PICRUSt2 repository under the GNU General Public License, verify the bundled source before redistribution or reuse.

## Related tools

- **PICRUSt2** (The parent command-line pipeline whose installation wiki and bundled example data drive the workflow; installation instructions are documented on its GitHub wiki) — https://github.com/picrust/picrust2
- **MinPath** (Pathway-inference component that performs minimum-parsimony reconstruction over KEGG/MetaCyc pathway databases to map KO abundances to pathway abundances) — https://github.com/picrust/picrust2

## Examples

```
picrust2_pipeline.py -i ko_abundances.tsv -o pipeline_out -p 4
```

## Evaluation signals

- path_abun.tsv exists in the working directory after execution
- path_abun.tsv is a valid tab-delimited table with pathway identifier and abundance columns
- path_abun.tsv is consistent with the reference pathway abundance output produced by the PICRUSt2 tutorial workflow
- Installation succeeded per the official wiki instructions at https://github.com/picrust/picrust2/wiki/Installation
- Pathway identifiers in the output correspond to MetaCyc/KEGG pathway databases used for reconstruction

## Limitations

- MinPath is distributed under the GNU General Public License, which constrains redistribution of the bundled source code
- No changelog was found in the source document, so version-specific behavior must be verified against the wiki documentation
- Users are directed to consult the PICRUSt2 wiki and FAQ before running; questions belong on the PICRUSt users Google group rather than GitHub issues

## Evidence

- [methods] Install PICRUSt2 following the official installation wiki instructions: "Installation instructions are found [here](https://github.com/picrust/picrust2/wiki/Installation)."
- [readme] MinPath bundled with PICRUSt2 under GPL: "Note that this repository contains the source code for MinPath, which is distributed under the GNU General Public License."
- [readme] Wiki documentation covers install, running, and internals: "There is a lot of information within this Wiki about how to install and run PICRUSt2 as well as how it works"
